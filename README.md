## Create a class definition with ASNA Visual RPG

This command line utility creates an ASNA Visual RPG class for a given DG file.

If you run CreateClassDefinition.exe without any command line arguments it displays this help:

```
Generate DataGate file class

Flag                  ShortHand  Required  Description
--------------------  ---------  --------  ---------------------------------------------
--databasename           -d        True    Database Name
--libraryname            -l        True    Library Name
--filename               -f        True    File Name
--classname              -c        True    Class Name
--help                   -h                Show this help
```

To create a class named `Customer` over the `Examples/CMastNew2` file using the `*Public/DG Net Local` database name, use this command line:

```
 createClassdefinition.exe -d "*Public/DG Net Local" -l Examples -f CMastNewL2 -c Customer
```

This creates a class as shown below. It contains public properties for each field in the file. It also contains one method named `GetInstanceFromDataRow` which is explained below.

```
BegClass CustomerModel Access(*Public)

    // This class was created with the CreateClassDefinition utility.
    // Do not change it by hand--regenerate it with the CreateClassDefinition utility.
    // Property names must be lowercase to correspond to AddGetInstanceFromDataRowMethod's
    // lowercase property name look up.

    DclProp cmcustno Type(System.Decimal) Access(*Public)
    DclProp cmname Type(System.String) Access(*Public)
    DclProp cmaddr1 Type(System.String) Access(*Public)
    DclProp cmcity Type(System.String) Access(*Public)
    DclProp cmstate Type(System.String) Access(*Public)
    DclProp cmcntry Type(System.String) Access(*Public)
    DclProp cmpostcode Type(System.String) Access(*Public)
    DclProp cmactive Type(System.String) Access(*Public)
    DclProp cmfax Type(System.Decimal) Access(*Public)
    DclProp cmphone Type(System.String) Access(*Public)

    BegFunc GetInstanceFromDataRow Type(Customer) Access(*Public) Shared(*Yes)
        DclSrParm dt Type(DataTable)

        DclFld ColumnName Type(*String)
        DclFld obj Type(CustomerModel) New()
        DclFld dr Type(DataRow)
        DclFld i Type(*Integer4)
        DclFld pi Type(PropertyInfo)
        DclFld Value Type(*Object)

        DclFld dc Type(DataColumn)

        dr = dt.Rows[0]

        For Index(i = 0) To(dt.Columns.Count - 1)
            dc = dt.Columns[i]
            ColumnName = dt.Columns[i].ColumnName
            Value = dr.ItemArray[i]

            pi = obj.GetType().GetProperty(ColumnName.ToLower())
            If pi <> *Nothing
                pi.SetValue(obj, Convert.ChangeType(Value, pi.PropertyType), *Nothing)
            EndIf
        EndFor

        LeaveSr obj
    EndFunc

EndClass
```

#### GetInstanceFromDataRow 

The `GetInstanceFromDataRow` method returns a instance of `CustomerModel` that is populated with the zeroth row of the `DataTable` argument. 

For example, the code below populates a `List(*Of CustomerModel)` with ten records read from a file. This code populates a row of a memory file based on the definition of the `DclDiskFile` object. The memory file is always cleared before a record is written to it--ensuring the current record is is always the zeroth record in the memory file. This does two things:

1. `GetInstanceFromDataRow` _always_ reads from the zeroth row in the incoming `DataTable` object.
2. It ensures that the memory file doesn't use unnecessary memory. 

```
    DclDB pgmDB DBName('*Public/DG NET Local') 
                
    DclDiskFile Customer +
        Type(*Input) + 
        Org(*Indexed) + 
        File("examples/cmastnewl2") +
        DB(pgmDB) +
        ImpOpen(*No)  

    DclMemoryFile CustomerMF +
        DBDesc("*Public/DG NET Local") +
        FileDesc("Examples/CMastNewL2") +
        RnmFmt(RMemFile)
    
    BegSr Run Access(*Public) 
        OpenData()
        ReadRows()
        CloseData()
    EndSr

    BegSr ReadRows
        DclFld cust Type(CustomerModel)
        DclFld Customers Type(List(*Of CustomerModel)) New() 

        Do FromVal(1) ToVal(10)
            Read Customer
            CustomerMF.DataSet.Tables[0].Clear()
            Write CustomerMF

            cust = CustomerModel.GetInstanceFromDataRow(CustomerMF.DataSet.Tables[0])
            Customers.Add(cust) 
        EndDo 
    EndSr         

    BegSr OpenData Access(*Public)
        If (NOT pgmDB.IsOpen)
            Connect pgmDB
        EndIf
        Open Customer
        Open CustomerMF
    EndSr

    BegSr CloseData Access(*Public)
        Close Customer
    EndSr
EndClass
```