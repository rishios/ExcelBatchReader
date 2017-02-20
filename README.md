ExcelBatchReader
================

Lightweight and fast library written in C# for reading Microsoft Excel files (.xlsx and .xls) in batch.
It does not requires excel or access installation.

Features:
* Batch reading, Schema reading, Row skipping, Row reading.
* Reading top rows, all rows, sheet names, column names, column data types.
* Finding blank sheets through GetSchema method.

## Finding the binaries
The compiled binaries are available in the release. To try out the features, download binary files (Binaries.zip)
and use in your project.

## How to use
### C# code :
```c#

using Excel;
...

//1. Reading sheet names
	List<string> sheetNames = null;
	using (IExcelDataReader excelReader =
	ExcelReaderFactory.CreateOpenXmlReader(Helper.GetTestWorkbook("xTestMultiSheet")))
	{
		if (!excelReader.IsValid) { throw new Exception(excelReader.ExceptionMessage); }
		sheetNames = excelReader.GetSheetNames();
	}
//2. Reading top rows
	DataTable dataTable = null;
    using (IExcelDataReader excelReader =
	ExcelReaderFactory.CreateOpenXmlReader(Helper.GetTestWorkbook("xTestMultiSheet")))
    {
        if (!excelReader.IsValid) { throw new Exception(excelReader.ExceptionMessage); }
        dataTable = excelReader.GetTopRows(5, new SheetParameters("Sheet2", false));
    }
//3. Reading schema (sheetNames, columnNames and dataTypes)
//	 Note: This forms schema based on first 1000 rows, to increase sample size for inferring schema
//   set excelReader.BatchSize of desired rows. 
	DataSet dataset = null;
	using (IExcelDataReader excelReader =
	ExcelReaderFactory.CreateOpenXmlReader(Helper.GetTestWorkbook("xTestMultiSheet")))
	{
		if (!excelReader.IsValid) { throw new Exception(excelReader.ExceptionMessage); }
		dataset = excelReader.GetSchema();
	}
//4. Read single sheet in batch
	DataTable dataTable = null;
	using (IExcelDataReader excelReader =
	ExcelReaderFactory.CreateOpenXmlReader(Helper.GetTestWorkbook("xTestMultiSheet")))
	{
		if (!excelReader.IsValid) { throw new Exception(excelReader.ExceptionMessage); }
		// Read one sheet of excel in batch
		excelReader.SheetName = "Sheet2";
		excelReader.IsFirstRowAsColumnNames = false; // default is true
		excelReader.SkipRows = 0; // default is 0
		excelReader.BatchSize = 10000; // modify as per need, default is 1000
		while (excelReader.ReadBatch())
		{
			dataTable = excelReader.GetCurrentBatch();
			// process batch rows
		}
	}
//5. Read all sheets of excel in batch
	DataSet dataSet = null;
	DataTable dataTable = null;
	using (IExcelDataReader excelReader =
	ExcelReaderFactory.CreateOpenXmlReader(Helper.GetTestWorkbook("xTestMultiSheet")))
	{
		if (!excelReader.IsValid) { throw new Exception(excelReader.ExceptionMessage); }
		dataSet = excelReader.GetSchema();
		excelReader.BatchSize = 10000; // modify as per need, default is 1000
		foreach (DataTable dt in dataSet.Tables)
		{
			excelReader.SheetName = dt.TableName;
			excelReader.IsFirstRowAsColumnNames =
			Convert.ToBoolean(dt.ExtendedProperties["IsFirstRowAsColumnNames"]);
			excelReader.SkipRows = Convert.ToInt32(dt.ExtendedProperties["SkipRows"]);
			while (excelReader.ReadBatch())
			{
				dataTable = excelReader.GetCurrentBatch();
				// process batch rows
			}
		}
	}

// The below methods of ExcelDataReader which is extended to ExcelBatchReader will work as expected.

using Excel;
...

FileStream stream = File.Open(filePath, FileMode.Open, FileAccess.Read);

//1. Reading from a binary Excel file ('97-2003 format; *.xls)
IExcelDataReader excelReader = ExcelReaderFactory.CreateBinaryReader(stream);

//2. Reading from a OpenXml Excel file (2007 format; *.xlsx)
IExcelDataReader excelReader = ExcelReaderFactory.CreateOpenXmlReader(stream);

//3. DataSet - The result of each spreadsheet will be created in the result.Tables
DataSet result = excelReader.AsDataSet();

//4. DataSet - Create column names from first row
excelReader.IsFirstRowAsColumnNames = true;
DataSet result = excelReader.AsDataSet();

//5. Data Reader methods
while (excelReader.Read())
{
	//excelReader.GetInt32(0);
}

//6. Free resources (IExcelDataReader is IDisposable)
excelReader.Close();

```

### Notes
* ExcelBatchReader is an extention of ExcelDataReader. A pull request is in-progress.
* Use multi using statement when using IExcelDataReader so that the excel file handle is closed properly.
  IExcelDataReader's dispose method does not close excel file handle immediately. Refer below code:
```c#
	using (FileStream fileStream = File.OpenRead(filePath))
	using (IExcelDataReader excelReader = ExcelReaderFactory.CreateOpenXmlReader(fileStream))
	{
		if (!excelReader.IsValid) { throw new Exception(excelReader.ExceptionMessage); }
		List<string> sheetNames = excelReader.GetSheetNames();
	}
```

