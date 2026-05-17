---
title: "SpecFlow with Coded UI vsts 2013"
date: 2016-01-05 05:50:39 
author: shajeebhameed
layout: page
slug: specflow-with-coded-ui-vsts-2013
status: publish
---

**Reference** : 

  * https://groups.google.com/forum/#!topic/specflow/WwOOPig_2_4
  * https://groups.google.com/forum/#!topic/specflow/e2-KZQhuOMM/discussion
  * http://www.specflow.org/documentation/Plugins/

**Need Packages**

  1. NUnit.2.6.0.12054
  2. SpecFlow.1.9.0
  3. SpecFlow.CustomPlugin.1.9.0
  4. SpecFlow.MsTest.1.0.0
  5. SpecFlow.NUnit.1.1.1

**MsTest2010CodedUiGeneratorProvider** public class MsTest2010CodedUiGeneratorProvider : MsTest2010GeneratorProvider { public MsTest2010CodedUiGeneratorProvider(CodeDomHelper codeDomHelper) : base(codeDomHelper) { } public override void SetTestClass(TechTalk.SpecFlow.Generator.TestClassGenerationContext generationContext, string featureTitle, string featureDescription) { base.SetTestClass(generationContext, featureTitle, featureDescription); foreach (CodeAttributeDeclaration customAttribute in generationContext.TestClass.CustomAttributes) { if (customAttribute.Name == "Microsoft.VisualStudio.TestTools.UnitTesting.TestClassAttribute") { generationContext.TestClass.CustomAttributes.Remove(customAttribute); break; } } generationContext.TestClass.CustomAttributes.Add(new CodeAttributeDeclaration(new CodeTypeReference("Microsoft.VisualStudio.TestTools.UITesting.CodedUITestAttribute"))); } } **MyTestingPlugin.SpecFlowPlugin** [assembly: GeneratorPlugin(typeof(GeneratorTmsPlugin))] namespace TMSTestingPlugin.SpecFlowPlugin { using BoDi; using System.CodeDom; using TechTalk.SpecFlow.Generator; using TechTalk.SpecFlow.Generator.Configuration; using TechTalk.SpecFlow.Generator.Plugins; using TechTalk.SpecFlow.Generator.UnitTestProvider; using TechTalk.SpecFlow.Utils; public class MsTestCustomGeneratorProvider : MsTest2010GeneratorProvider { /// <summary> /// Initializes a new instance of the <see cref="MsTestCustomGeneratorProvider"/> class. /// </summary> /// <param name="codeDomHelper"> /// The code dom helper. /// </param> public MsTestCustomGeneratorProvider(CodeDomHelper codeDomHelper) : base(codeDomHelper) { } /// <summary> /// The finalize test class. /// </summary> /// <param name="generationContext"> /// The generation context. /// </param> public override void FinalizeTestClass(TestClassGenerationContext generationContext) { base.FinalizeTestClass(generationContext); var msTestContextGeneration = new CodeExpressionStatement( new CodeMethodInvokeExpression( new CodeTypeReferenceExpression("testRunner.ScenarioContext"), "Add", new CodeExpression[] { new CodePrimitiveExpression("MSTestContext"), new CodeArgumentReferenceExpression("TestContext") })); generationContext.ScenarioInitializeMethod.Statements.Add(msTestContextGeneration); var msTestAssignment = new CodeAssignStatement( new CodeVariableReferenceExpression("TmsTestContext"), new CodeVariableReferenceExpression("testContext")); generationContext.TestClassInitializeMethod.Statements.Add(msTestAssignment); var mstestContextFiled = new CodeMemberField { Attributes = MemberAttributes.Private | MemberAttributes.Static | MemberAttributes.Final, Name = "TmsTestContext", Type = new CodeTypeReference(TESTCONTEXT_TYPE), }; generationContext.TestClass.Members.Add(mstestContextFiled); var mstestContextProperty = new CodeMemberProperty(); mstestContextProperty.Name = "TestContext"; mstestContextProperty.Type = new CodeTypeReference(TESTCONTEXT_TYPE); mstestContextProperty.Attributes = (mstestContextProperty.Attributes & ~MemberAttributes.AccessMask) | MemberAttributes.Public; mstestContextProperty.GetStatements.Add(new CodeMethodReturnStatement(new CodeVariableReferenceExpression("TmsTestContext"))); mstestContextProperty.SetStatements.Add(new CodeAssignStatement(new CodeVariableReferenceExpression("TmsTestContext"), new CodePropertySetValueReferenceExpression())); generationContext.TestClass.Members.Add(mstestContextProperty); } } public class GeneratorTmsPlugin : IGeneratorPlugin { public void RegisterDependencies(ObjectContainer container) { } public void RegisterCustomizations(ObjectContainer container, SpecFlowProjectConfiguration generatorConfiguration) { container.RegisterTypeAs<MsTestCustomGeneratorProvider, IUnitTestGeneratorProvider>(); } public void RegisterConfigurationDefaults(SpecFlowProjectConfiguration specFlowConfiguration) { } } }
