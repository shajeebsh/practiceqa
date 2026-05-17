---
title: "Excel Automation C#"
date: 2016-05-27 16:22:48 
author: shajeebhameed
layout: page
slug: excel-automation-c
status: publish
---

=====================WsPerfMonCCC====================== 

public class WsPerfMonCCC : BaseWsHandler

{

private static string HEADING_COLUMN = "PERFORMANCE MONITOR - CONFERENCE CONTROL CENTER (MEMORY, PHYSICAL DISK and PROCESSOR STATISTICS)";

Excel.Worksheet _sheet = null;

List<DataSet> _dsAllScenario = null;

int _lastColPosition = 1;

string _memoryChartRange = string.Empty;

string _processorChartRange = string.Empty;

string _processChartRange = string.Empty;

string _physicalDiskChartRange = string.Empty;

string _threadCountChartRange = string.Empty;

string _testChartRange = string.Empty;

public override void HandleRequest(WsType type)

{

if (type == WsType.PerfMonCCC)

{

ProcessWS();

}

else if (base._successor != null)

{

base._successor.HandleRequest(type);

}

}

private void ProcessWS()

{

InitializePerfMonCCC();

UpdatePerfMonCCCExcel();

}

private void InitializePerfMonCCC()

{

_sheet = (Excel.Worksheet)_xlApp.Sheets[WsType.PerfMonCCC.ToString()];

_dsAllScenario = new List<DataSet>();

if (__dsPerfData == null || __dsPerfData.Count == 0)

{

__dsPerfData = new List<System.Data.DataTable>();

DirectoryInfo dirInfo = new DirectoryInfo(Config.PerfMonitorOutputPath);

var files = dirInfo.GetFiles().Where(f => f.Name.Contains("PerfMonitorFinalOutput"));

foreach (var file in files)

{

var dt = new CsvUtil(file.FullName).GetData().Tables[0];

dt.TableName = file.Name.Remove(file.Name.IndexOf('.'));

__dsPerfData.Add(dt);

}

}

foreach (System.Data.DataTable dtTrialData in __dsPerfData)

{

ProcessPerfData(dtTrialData);

}

}

private void UpdatePerfMonCCCExcel()

{

bool IsHeadingDone = false;

for (int i = 0; i < _dsAllScenario.Count(); i++)

{

var dsData = _dsAllScenario[i];

UpdatePerfMonCCC_Data(dsData, i, IsHeadingDone); // TO DO: this can be reusable for ListConf and Navigator

}

CreateChart(10, 500, 325, 250, "Available Bytes Vs Pages/Sec", "Bytes",

"300/600/1500/3000/5000 Non Recurrent and Recurrent Runs", _sheet, _memoryChartRange);

CreateChart(350, 500, 325, 250, "Processor", "Bytes",

"300/600/1500/3000/5000 Non Recurrent and Recurrent Runs", _sheet, _processorChartRange);

CreateChart(10, 770, 325, 250, "Process", "Bytes",

"300/600/1500/3000/5000 Non Recurrent and Recurrent Runs", _sheet, _processChartRange);

CreateChart(350, 770, 325, 250, "Physical Disk", "Bytes",

"300/600/1500/3000/5000 Non Recurrent and Recurrent Runs", _sheet, _physicalDiskChartRange);

CreateChart(10, 1040, 325, 250, "Thread Count", "Thread Count Number",

"300/600/1500/3000/5000 Non Recurrent and Recurrent Runs", _sheet, _threadCountChartRange);

CreateChart(350, 1040, 325, 250, "Test", "<< Test test >>",

"<<< TEST >>>", _sheet, _testChartRange);

}

private void ProcessPerfData(System.Data.DataTable dtData)

{

__scenarios = new List<string>();

foreach (DataColumn col in dtData.Columns) // TO DO: _scenarios Can be make one time.

{

if (!col.ColumnName.Contains(CATEGORY_COLUMN) && !col.ColumnName.Contains(COUNTER_COLUMN) && !__scenarios.Contains(col.ColumnName.Split('_')[0]))

__scenarios.Add(col.ColumnName.Split('_')[0]);

}

CategorizeData(dtData);

}

private void CategorizeData(System.Data.DataTable dtData)

{

System.Data.DataTable scenarioDt = null;

System.Data.DataSet scenarioTables = new DataSet();

foreach (var scenario in __scenarios)

{

scenarioDt = dtData.Copy();

scenarioDt.TableName = scenario;

foreach (DataColumn col in dtData.Columns)

{

if (!col.ColumnName.Contains(scenario) && !col.ColumnName.Contains(CATEGORY_COLUMN) && !col.ColumnName.Contains(COUNTER_COLUMN))

{

scenarioDt.Columns.Remove(col.ColumnName);

}

}

scenarioDt.Columns.Add(AVERAGE_COLUMN);

CleanTableValues(scenarioDt);

scenarioTables.Tables.Add(scenarioDt);

scenarioTables.DataSetName = dtData.TableName;

}

_dsAllScenario.Add(scenarioTables);

}

private void UpdatePerfMonCCC_Data(System.Data.DataSet scenarioTables, int tableIndex, bool IsHeadingDone)

{

System.Data.DataTable dtCCC = scenarioTables.Tables[WsScenarioType.CCC.ToString()];

int columnsCount = dtCCC.Columns.Count;

int rowCount = dtCCC.Rows.Count;

int headerColStart = _lastColPosition, headerColEnd = columnsCount;

if (tableIndex > 0)

{

dtCCC.Columns.Remove(CATEGORY_COLUMN);

dtCCC.Columns.Remove(COUNTER_COLUMN);

columnsCount = dtCCC.Columns.Count;

headerColEnd = headerColStart + columnsCount - 1;

}

_lastColPosition = _lastColPosition + columnsCount;

int sectionHeadStart = headerColStart;

if (tableIndex.Equals(0))

sectionHeadStart = sectionHeadStart + 2;

Range headerRange = null;

if (!IsHeadingDone)

{

headerRange = (Range)(_sheet.Cells[2, COUNTER_HEADING]);

headerRange.Value = HEADING_COLUMN;

headerRange.Font.Bold = true;

headerRange = _sheet.get_Range((Range)_sheet.Cells[3, sectionHeadStart], (Range)(_sheet.Cells[3, headerColEnd]));

headerRange.Merge();

headerRange.Value = scenarioTables.DataSetName.Split('_')[0];

headerRange.Interior.Color = System.Drawing.ColorTranslator.ToOle(System.Drawing.Color.LightGreen);

headerRange.Borders.LineStyle = Excel.XlLineStyle.xlContinuous;

headerRange.HorizontalAlignment = Excel.XlHAlign.xlHAlignCenter;

headerRange = (Range)(_sheet.Cells[4, headerColEnd]);

headerRange.EntireColumn.Font.Bold = true;

IsHeadingDone = true;

}

var columnNames = base.GetColumnObjectArray(dtCCC);

headerRange = _sheet.get_Range((Range)_sheet.Cells[4, headerColStart], (Range)(_sheet.Cells[4, headerColEnd]));

headerRange.Value = columnNames;

headerRange.Interior.Color = System.Drawing.ColorTranslator.ToOle(System.Drawing.Color.LightBlue);

headerRange.Font.Bold = true;

var cellValues = base.GetRowObjectArray(dtCCC);

Range rowRange = _sheet.get_Range((Range)_sheet.Cells[5, headerColStart], (Range)(_sheet.Cells[rowCount + 1, headerColEnd]));

rowRange.Value = cellValues;

rowRange.Columns.AutoFit();

_memoryChartRange = UpdateChartRangeY(headerColEnd, _memoryChartRange, 30, 31, _sheet);

_processChartRange = UpdateChartRangeY(headerColEnd, _processChartRange, 6, 23, _sheet);

_processorChartRange = UpdateChartRangeY(headerColEnd, _processorChartRange, 5, 5, _sheet);

_physicalDiskChartRange = UpdateChartRangeY(headerColEnd, _physicalDiskChartRange, 24, 29, _sheet);

_threadCountChartRange = UpdateChartRangeY(headerColEnd, _threadCountChartRange, 20, 20, _sheet);

_testChartRange = UpdateChartRangeY(headerColEnd, _testChartRange,

new List<RangePairY>() { new RangePairY() { RowValStart = 16,RowValEnd = 17 },

new RangePairY() { RowValStart = 20, RowValEnd = 22 },

new RangePairY() { RowValStart = 11, RowValEnd = 15 },

new RangePairY() { RowValStart = 25, RowValEnd = 30 }

}, _sheet);

}

}

  =====================WsMain ====================== 

public class WsMain : BaseWsHandler

{

public WsMain()

{

base.__sheetNames = new List<WsType>();

base.__sheetNames.Add(WsType.Main);

base.__sheetNames.Add(WsType.PerfMonCCC);

base.__sheetNames.Add(WsType.PerfMonListConf);

base.__sheetNames.Add(WsType.PerfMonNavigator);

base.__sheetNames.Add(WsType.SqlProfilerCCC);

base.__sheetNames.Add(WsType.SqlProfilerListConf);

base.__sheetNames.Add(WsType.SqlProfilerNavigator);

base.__sheetNames.Add(WsType.WebProfilerCCC);

base.__sheetNames.Add(WsType.WebProfilerListConf);

base.__sheetNames.Add(WsType.WebProfilerNavigator);

base.__sheetNames.Add(WsType.EP_Recurrent300);

base.__sheetNames.Add(WsType.EP_Recurrent600);

base.__sheetNames.Add(WsType.EP_Recurrent1500);

base.__sheetNames.Add(WsType.EP_Recurrent3000);

base.__sheetNames.Add(WsType.EP_Recurrent5000);

base.__sheetNames.Add(WsType.EP_Non_R

=====================WsPerfMonCCC======================

public class WsPerfMonCCC : BaseWsHandler

{

private static string

ecurrent300);

base.__sheetNames.Add(WsType.EP_Non_Recurrent600);

base.__sheetNames.Add(WsType.EP_Non_Recurrent1500);

base.__sheetNames.Add(WsType.EP_Non_Recurrent3000);

base.__sheetNames.Add(WsType.EP_Non_Recurrent5000);

base.__sheetNames.Add(WsType.Screenshots);

base.__sheetNames.Reverse();

}

public override void HandleRequest(WsType type)

{

if (type == WsType.Main)

{

ProcessWS();

}

else if (base._successor != null)

{

base._successor.HandleRequest(type);

}

}

public void ProcessWS()

{

CreateWorkSheets();

CreateMainSheetIndex();

CreateBackLinkInSheet();

}

private void CreateBackLinkInSheet()

{

int index = 0;

Excel.Range rangeBackLink;

foreach (Excel.Worksheet worksheet in _xlApp.Worksheets)

{

if (worksheet.Name.Equals("Main"))

continue;

rangeBackLink = worksheet.get_Range("B1");

rangeBackLink.Cells.Hyperlinks.Add(rangeBackLink, string.Format("#'Main'!A1", Type.Missing),

Type.Missing, Type.Missing, "<<< Back To Main Page");

rangeBackLink.Style.Font.Name = "Verdana";

rangeBackLink.Style.Font.Bold = true;

rangeBackLink.Style.Font.Size = 10;

rangeBackLink.Style.Font.Color = System.Drawing.ColorTranslator.ToOle(System.Drawing.Color.Green);

rangeBackLink.Borders.Color = System.Drawing.ColorTranslator.ToOle(System.Drawing.Color.Blue);

rangeBackLink.Interior.ColorIndex = 36;

//rangeBackLink.Columns.AutoFit();

index++;

}

}

private void CreateMainSheetIndex()

{

int index = 0;

// Create an array to multiple values at once.

string[,] colHeading = new string[1, 2];

colHeading[0, 0] = "SlNo";

colHeading[0, 1] = "Sheet Name";

_xlApp.get_Range("B2", "C2").Value2 = colHeading;

_xlApp.get_Range("B2", "C2").Font.Bold = true;

Excel.Range rangMainWorkSheet = _xlApp.get_Range("C3");

foreach (Excel.Worksheet worksheet in _xlApp.Worksheets)

{

if (worksheet.Name.Equals("Main"))

continue;

rangMainWorkSheet.Cells.Hyperlinks.Add(rangMainWorkSheet.get_Offset(index, 0), string.Format("#'{0}'!A1", worksheet.Name),

Type.Missing, worksheet.Name, worksheet.Name);

index++;

}

rangMainWorkSheet.EntireColumn.AutoFit();

}

private void CreateWorkSheets()

{

Excel._Workbook xlWB;

try

{

_xlApp = new Excel.Application();

xlWB = (Excel._Workbook)(_xlApp.Workbooks.Add(Missing.Value));

((Excel._Worksheet)xlWB.ActiveSheet).Name = __sheetNames.First().ToString();

for (int i = 1; i < __sheetNames.Count; i++)

{

xlWB.Worksheets.Add(Missing.Value);

xlWB.Worksheets[1].Name = __sheetNames[i].ToString();

}

_xlApp.Visible = true;

_xlApp.UserControl = true;

}

catch (Exception theException)

{

throw theException;

}

}

}

  =====================BaseWsHandler====================== 

public abstract class BaseWsHandler

{

protected BaseWsHandler _successor;

protected List<WsType> __sheetNames;

protected static Excel.Application _xlApp;

protected static List<string> __scenarios;

protected static List<System.Data.DataTable> __dsPerfData;

protected static List<System.Data.DataTable> __dsSqlData;

protected static List<System.Data.DataTable> __dsWebProfilerData;

protected static int COUNTER_HEADING = 2;

protected static string CATEGORY_COLUMN = "Category";

protected static string COUNTER_COLUMN = "Counters";

protected static string AVERAGE_COLUMN = "Average";

public List<WsType> SheetNames { get { return __sheetNames; } }

public void SetSuccessor(BaseWsHandler successor)

{

this._successor = successor;

}

public abstract void HandleRequest(WsType type);

protected object[] GetColumnObjectArray(System.Data.DataTable data)

{

return data.Columns.Cast<System.Data.DataColumn>().Select(column => column.ColumnName as object).ToArray();

}

protected object[,] GetRowObjectArray(System.Data.DataTable data)

{

var cellValues = Array.CreateInstance(typeof(object), data.Rows.Count, data.Columns.Count) as object[,];

for (int j = 0; j < data.Rows.Count; j++)

for (int i = 0; i < data.Columns.Count; i++)

cellValues[j, i] = data.Rows[j][i];

return cellValues;

}

protected string GetCellAddress(Excel.Range range)

{

return range.get_AddressLocal(false, false, Excel.XlReferenceStyle.xlA1, Type.Missing, Type.Missing);

}

protected void CreateChart(int left, int top, int width, int height,

string chartTitle, string xAxisTitle, string yAxisTitle, Excel.Worksheet currentSheet, string chartRange)

{

Excel.Range chartRangePointer;

Excel.ChartObjects xlCharts = (Excel.ChartObjects)currentSheet.ChartObjects(Type.Missing);

Excel.ChartObject chartObject = (Excel.ChartObject)xlCharts.Add(left, top, width, height);

Excel.Chart chartControl = chartObject.Chart;

chartRangePointer = currentSheet.get_Range(chartRange);

chartControl.SetSourceData(chartRangePointer, Type.Missing);

chartControl.ChartType = Excel.XlChartType.xlColumnClustered;

chartControl.ChartStyle = 2;

// Main title:

chartControl.HasTitle = true;

chartControl.ChartTitle.Text = chartTitle;

// X Axis title:

var xAxis = (Excel.Axis)chartControl.Axes(Excel.XlAxisType.xlValue, Excel.XlAxisGroup.xlPrimary);

xAxis.HasTitle = true;

xAxis.AxisTitle.Text = xAxisTitle;

// Y Axis title:

var yAxis = (Excel.Axis)chartControl.Axes(Excel.XlAxisType.xlCategory, Excel.XlAxisGroup.xlPrimary);

yAxis.HasTitle = true;

yAxis.AxisTitle.Text = yAxisTitle;

}

protected string UpdateChartRangeY(int headerColEnd, string range, int startRowVal, int endRowVal, Excel.Worksheet currentSheet)

{

string headingColumn = GetCellAddress(currentSheet.Cells[startRowVal, COUNTER_HEADING]);

if (string.IsNullOrEmpty(range))

{

range = string.Format("{0}:{1},{2}:{3}", headingColumn, GetCellAddress(currentSheet.Cells[endRowVal, COUNTER_HEADING]),

GetCellAddress(currentSheet.Cells[startRowVal, headerColEnd]), GetCellAddress(currentSheet.Cells[endRowVal, headerColEnd]));

}

else if (!range.Contains(headingColumn))

{

range += string.Format(",{0}:{1},{2}:{3}", headingColumn, GetCellAddress(currentSheet.Cells[endRowVal, COUNTER_HEADING]),

GetCellAddress(currentSheet.Cells[startRowVal, headerColEnd]), GetCellAddress(currentSheet.Cells[endRowVal, headerColEnd]));

}

else

range += string.Format(",{0}:{1}", GetCellAddress(currentSheet.Cells[startRowVal, headerColEnd]), GetCellAddress(currentSheet.Cells[endRowVal, headerColEnd]));

return range;

}

protected string UpdateChartRangeY(int headerColEnd, string range, List<RangePairY> ranges, Excel.Worksheet currentSheet)

{

foreach (var rangeItem in ranges)

range = UpdateChartRangeY(headerColEnd, range, rangeItem.RowValStart, rangeItem.RowValEnd, currentSheet);

return range;

}

protected string UpdateChartRangeX(string range, RangePairX cell, Excel.Worksheet currentSheet)

{

if (string.IsNullOrEmpty(range))

range = string.Format("{0}:{1}", GetCellAddress(currentSheet.Cells[cell.RowValStart, cell.ColIndex]),

GetCellAddress(currentSheet.Cells[cell.RowValEnd, cell.ColIndex]));

else

range += string.Format(",{0}:{1}", GetCellAddress(currentSheet.Cells[cell.RowValStart, cell.ColIndex]),

GetCellAddress(currentSheet.Cells[cell.RowValEnd, cell.ColIndex]));

return range;

}

protected string UpdateChartRangeX(string range, List<RangePairX> ranges, Excel.Worksheet currentSheet)

{

foreach (var rangeItem in ranges)

range = UpdateChartRangeX(range, rangeItem, currentSheet);

return range;

}

/// <summary>

/// Cleaning up of Values - for WsPerfMonCCC, WsPerfMonListConf and WsPerfMonNavigator

/// </summary>

/// <param name="scenarioDt"></param>

protected static void CleanTableValues(System.Data.DataTable scenarioDt)

{

foreach (DataColumn col in scenarioDt.Columns)

{

if (col.ColumnName.Contains("Trial"))

{

string colname = col.ColumnName;

col.ColumnName = colname.Substring(colname.IndexOf("Trial"), (colname.IndexOf("#") - colname.IndexOf("Trial")));

}

}

foreach (DataRow row in scenarioDt.Rows)

{

row[CATEGORY_COLUMN] = row[CATEGORY_COLUMN].ToString().Replace("(_Total)", "");

List<double> vals = new List<double>();

for (int i = 2; i < scenarioDt.Columns.Count - 1; i++)

vals.Add(double.Parse(row[i].ToString()));

row[AVERAGE_COLUMN] = Math.Round(vals.Average(), 3);

}

}

}
