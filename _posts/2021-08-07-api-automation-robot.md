---
title: "API Automation - Robot"
date: 2021-08-07 07:39:16 
author: shajeebhameed
layout: page
slug: api-automation-robot
status: publish
---

API Testing Using RobotFramework
    
    
    P**ath: ../Library.robot**
    
    *** Settings ***
    Documentation         API Testing Keywords for the operations like GET, PUT, POST, PATCH and DELETE. This library is based on RequestsLibrary.
    Library               Collections
    Library               RequestsLibrary
    
    *** Keywords ***
    
    Create HTTP Session With No Authentication
        [Documentation]  Creates session for performing HTTP actions
        [Arguments]  ${url}  ${alias}
        Create Session  ${alias}  ${url}  auth=None
    
    Create HTTP Session With Authentication
        [Documentation]  Creates session for performing HTTP actions with credentials
        [Arguments]  ${url}  ${alias}  ${username}  ${password}
        ${auth}=    Create Dictionary   ${username}  ${password}
        Create Session  ${alias}  ${url}  auth=${auth}
    
    Fetch Authentication Token And Instance URL
        [Documentation]  Fetches authentication token and instance URL from Salesforce Sandbox environment
        [Arguments]  ${url}  ${alias}  ${grant_type}  ${client_id}  ${client_secret}  ${username}  ${password}
        ${headers}=     Create Dictionary       Content-Type=application/x-www-form-urlencoded
        ${data}=        Create Dictionary       grant_type=${grant_type}
                        ...     client_id=${client_id}
                        ...     client_secret=${client_secret}
                        ...     username=${username}
                        ...     password=${password}
        Create HTTP Session With No Authentication  ${url}  ${alias}
        ${response}=    Post Method     /services/oauth2/token  ${alias}  ${headers}  ${data}
        ${AccessToken}=    Evaluate    $response.json().get("access_token")
        ${InstanceUrl}=    Evaluate    $response.json().get("instance_url")
        Set Global Variable  ${AccessToken}
        Set Global Variable  ${InstanceUrl}
    
    Get Headers With Bearer Token
        [Documentation]  Creates headers with Bearer Token
        ${BearerToken}=     Catenate  Bearer  ${AccessToken}
        ${headers}=         Create Dictionary       Content-Type=application/json
                                        ...         Accept-Encoding=gzip
                                        ...         Authorization=${BearerToken}
        [Return]  ${headers}
    
    Post Method
        [Documentation]  Invokes POST Request from RequestKeywords library
        [Arguments]     ${path}  ${alias}  ${headers}  ${data}
        ${response}=     Post Request    ${alias}  ${path}  data=${data}  headers=${headers}
        Should Be Equal As Strings  ${response.status_code}     200
        [Return]  ${response}
    
    Get Method
        [Documentation]  Invokes GET Request from RequestKeywords library
        [Arguments]     ${path}  ${alias}
        ${headers}=     Get Headers With Bearer Token
        ${response}=     Get Request  ${alias}  ${path}  headers=${headers}
        [Return]  ${response}
    
    Put Method
        [Documentation]  Invokes PUT Request from RequestKeywords library
        [Arguments]     ${path}  ${alias}  ${data}
        ${headers}=     Get Headers With Bearer Token
        ${response}=     Put Request  ${alias}  ${path}  headers=${headers}  data=${data}
        [Return]  ${response}
    
    Patch Method
        [Documentation]  Invokes PATCH Request from RequestKeywords library
        [Arguments]     ${path}  ${alias}  ${data}
        ${headers}=     Get Headers With Bearer Token
        ${response}=     Patch Request  ${alias}  ${path}  headers=${headers}  data=${data}
        [Return]  ${response}
    
    Delete Method
        [Documentation]  Invokes DELETE Request from RequestKeywords library
        [Arguments]     ${path}  ${alias}
        ${headers}=     Get Headers With Bearer Token
        ${response}=     Delete Request  ${alias}  ${path}  headers=${headers}
        [Return]  ${response}
    

../ApiTest/Support.robot
    
    
    *** Settings ***
    Documentation           Support For YOUR-TTTTT Tests
    Resource                ../Library.robot
    
    *** Variables ***
    ${Base Url}                 https://test.salesforce.com
    ${alias}                    YOUR-TTTTT
    ${grant_type}               password
    ${client_id}                Get from Salesforce sandbox
    ${client_secret}            Get from Salesforce sandbox
    ${username}                 userid@yourmailid.com
    ${password}                 yourpasssword
    *** Keywords ***
    
    Initialize API Testing
        [Documentation]    Initialize API Testing by fetching Authentication token and Instance URL
        Fetch authentication token and instance URL      ${Base Url}     ${alias}
            ...     ${grant_type}     ${client_id}    ${client_secret}    ${username}     ${password}
        Should Not Be Empty       ${AccessToken}
        Should Not Be Empty       ${InstanceUrl}
    
    I create a session for HTTP request
        Create HTTP Session With No Authentication     ${InstanceUrl}      ${alias}
    
    Sent GET Request
        [Arguments]     ${path}
        ${response}=    Get Method  ${path}  ${alias}
        [Return]  ${response}
    
    I sent GET request with valid eExternalAppId that has no child case
        ${response}=    Sent GET Request    /services/apexrest/EMarketPlace/GetCases/?eMarketPlaceReqId=eMarketplaceReqId
        Should Be Equal As Strings          ${response.reason}  OK
        Should Be Equal As Strings          ${response.status_code}     200
        Set Test Variable                 ${response}
    
    I expect to receive case with no child cases
        ${json_dict}=    Evaluate    json.loads('''${response.content}''')   modules=json
        ${childCases} =    Set Variable   ${json_dict['cases'][1]['childCases']}
        Should Be Equal            ${childCases}      ${None}
        FOR    ${item}   IN    @{json_dict['cases']}
            Log    CaseNumber: ${item['caseNumber']}, ChildCases: ${item['childCases']}
        END
    
    I sent GET request with valid eMarketplaceId that has child cases
        ${response}=    Sent GET Request    /services/apexrest/EMarketPlace/GetCases/?eMarketPlaceReqId=eMarketplaceReqId2
        Should Be Equal As Strings          ${response.reason}  OK
        Should Be Equal As Strings          ${response.status_code}     200
        Set Test Variable                 ${response}
    
    I expect to receive case that has child cases
        ${json_dict}=    Evaluate    json.loads('''${response.content}''')   modules=json
        ${childCases}= Set Variable   ${json_dict['cases'][0]['childCases']}
        ${childCasesCount}=     Get Length  ${childCases}
        Should Be True ${childCasesCount}  > 0
    
    I sent GET request with valid eMarketplaceId that has Parent Case
        ${response}=    Sent GET Request    /services/apexrest/EMarketPlace/GetCases/?eMarketPlaceReqId=eMarketplaceReqId
        Should Be Equal As Strings          ${response.reason}  OK
        Should Be Equal As Strings          ${response.status_code}     200
        Set Test Variable                 ${response}
    
    I expect to receive case that has Parent Case
        ${json_dict}=    Evaluate    json.loads('''${response.content}''')   modules=json
        ${parentCase} =    Set Variable   ${json_dict['cases'][0]['parentCase']}
        Should Not Be Equal            ${parentCase}      ${None}
    
    
    I sent GET request with Requisition Id having No Cases
        ${response}=    Sent GET Request    /services/apexrest/EMarketPlace/GetCases/?eMarketPlaceReqId=trefsfsfs
        Set Test Variable                 ${response}
    
    I expect to receive Error Code 404
        Should Be Equal As Strings          ${response.status_code}     404
    
    I sent GET request with Invalid Or Blank Requisition Id
        ${response}=    Sent GET Request    /services/apexrest/EMarketPlace/GetCases/?eMarketPlaceReqId=test%23
        Set Test Variable                 ${response}
    
    I expect to receive Error Code 400
        Should Be Equal As Strings          ${response.status_code}     400

../ApiTest/Test.robot 
    
    
    *** Settings ***
    Documentation                   Jira-ID: YOUR-TTTTT
    Resource                        Support.robot
    Suite Setup                      Initialize API Testing
    
    *** Test Cases ***
    
    GET_Verify Case with no child cases
        [Documentation]  Verify Case with no child cases.
        [Tags]          TagApiTest
        Given I create a session for HTTP request
        When I sent GET request with valid eMarketplaceId that has no child case
        Then I expect to receive case with no child cases
    
    GET_Verify Case that has child cases
        [Documentation]  Verify Case that has child cases.
        [Tags]          TagApiTest
        Given I create a session for HTTP request
        When I sent GET request with valid eMarketplaceId that has child cases
        Then I expect to receive case that has child cases
    
    GET_Verify Case that has Parent Case
        [Documentation]  Verify Case that has Parent Case.
        [Tags]          TagApiTest
        Given I create a session for HTTP request
        When I sent GET request with valid eMarketplaceId that has Parent Case
        Then I expect to receive case that has Parent Case
    
    GET_Verify No Cases Found for the Requisition Id
        [Documentation]  Verify No Cases Found for the Requisition Id.
        [Tags]          TagApiTest
        Given I create a session for HTTP request
        When I sent GET request with Requisition Id having No Cases
        Then I expect to receive Error Code 404
    
    GET_Verify Invalid Or Blank Requisition Id
        [Documentation]  Verify Invalid Or Blank Requisition Id.
        [Tags]          TagApiTest
        Given I create a session for HTTP request
        When I sent GET request with Invalid Or Blank Requisition Id
        Then I expect to receive Error Code 400
