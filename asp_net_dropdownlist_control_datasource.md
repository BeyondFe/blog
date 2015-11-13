title: Asp.net Dropdownlist Control Datasource Expression
date: 2015-11-13
---

I'm fairly new to .NET and I'm finding it a challenge locating solutions for some problems I was having. I decided to start writing down the solutions I found or created. There might be better ways to solve the problems. If you find a better way please let me know.

Now on the problem. To start off the application is written in VB.NET 2.0. There is a form with a asp dropdown control that I needed to set the datasource. This dropdown control lives inside a  asp repeater control. I wanted to set the datasource inside an expression like the following:

```html
<asp:DropDownList runat="server" DataSource='<%# CreateDataSource() #%>' />
```

<!-- more -->

The code behind would have a function that creates a DataTable and returns it to be used as the datasource. The DataTable would have two columns, MyText and MyValue, which would be used for the name and value for the dropdownlist control.

```vbnet
Protected Function CreateDataSource() As DataTable
{
	'Create table to store data for the DropDownList control 
	Dim dt As DataTable = New DataTable()

	'Define the columns of the table
	dt.Columns.Add(New DataColumn("MyText", GetType(String))
	dt.Columns.Add(New DataColumn("MyValue", GetType(String))

	'Add rows to the DataTable

	Return dt
}
```

The issue was binding the name (what is visibly displayed) and value to be used in the dropdown control. I couldn't do this in the function above becuase it's job is to return a datasource and I didn't have a reference to the dropdown control. There are events you can hook into for the dropdownlist control like [OnInit](https://msdn.microsoft.com/en-us/library/system.web.ui.control.init.aspx?cs-save-lang=1&cs-lang=vb) where you can do this but I was hesitant becuase there didn't seem to be a way to pass paramters. Finally I found you can bind the name and value within the markup!

```html
<asp:DropDownList
	runat="server"
	DataSource='<%# CreateDataSource() #%>'
	DataTextField="MyText"
	DataValueField="MyValue"
 />
```

This worked perfectly and I liked how I can set values within the markup of the control.

