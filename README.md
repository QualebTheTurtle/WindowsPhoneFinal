# Windows Phone Development SQLite Tutorial Morosko Final

## Intro to SQLite

### Overview

SQLite is a popular low resource easy to use relational database. In this tutorial you will create a Windows Phone application that uses this database.

### Lets Get to It

#### Setup

* Create a new Pivotproject and name it SQLiteDemo.
* Lets start by adding the nessisary libraries and packages to our project. First click on tools then extentions and updates.

![alt text][img1]

* On the left panel in the extentions and updates window click online then SDK.
* In the search bar on the top left type SQLite and install the SQLite for windows phone sdk.

![alt text][img2]

* Next we need to add the sdk to our project. In the solution explorer right click the refrences and choose add refrence.

![alt text][img3]

* In the left pane of the refrence manager window choose windows phone then extentions and ensure the sqlite sdk is checked. Click okay.

![alt text][img4]

* Before moving on we need to change the platform to x86 to avoid errors. First right click the project in solution explorer, then select configuration manager. In the manager change the platform dropdown to x86 and select okay.

![alt text][img5]

![alt text][img6]

* Lets add the needed packages using the package manager. In the package manager window search for SQLitePCL then install the Portable Class Library for SQLite package. Then search wptoolkit and install the windows phone toolkit.

![alt text][img7]

#### UI

This is the Xaml for the user interface.

```xaml

<phone:PhoneApplicationPage
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:phone="clr-namespace:Microsoft.Phone.Controls;assembly=Microsoft.Phone"
    xmlns:shell="clr-namespace:Microsoft.Phone.Shell;assembly=Microsoft.Phone"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:toolkit="clr-namespace:Microsoft.Phone.Controls;assembly=Microsoft.Phone.Controls.Toolkit"
    x:Class="SQLiteDB.MainPage"
    mc:Ignorable="d"
    d:DataContext="{d:DesignData SampleData/MainViewModelSampleData.xaml}"
    FontFamily="{StaticResource PhoneFontFamilyNormal}"
    FontSize="{StaticResource PhoneFontSizeNormal}"
    Foreground="{StaticResource PhoneForegroundBrush}"
    SupportedOrientations="Portrait"  Orientation="Portrait"
    shell:SystemTray.IsVisible="True">

    <!--LayoutRoot is the root grid where all page content is placed-->
    <Grid x:Name="LayoutRoot" Background="Transparent">

        <phone:Pivot Title="SQLite Test">
            <!--Pivot item one-->
            <phone:PivotItem Header="Insert">
                <Grid>
                    <TextBlock x:Name="lbl" HorizontalAlignment="Left" Margin="23,37,0,0" TextWrapping="Wrap" Text="Name:" VerticalAlignment="Top"/>
                    <TextBox x:Name="txtInsert" HorizontalAlignment="Left" Height="72" Margin="0,64,0,0" TextWrapping="Wrap" VerticalAlignment="Top" Width="456"/>
                    <Button x:Name="btnInsert" Content="Insert" HorizontalAlignment="Left" Margin="328,141,0,0" VerticalAlignment="Top" Click="btnInsert_Click"/>
                </Grid>
            </phone:PivotItem>

            <!--Pivot item two-->
            <phone:PivotItem Header="Update" Margin="10,0,14,28">
                <Grid>
                    <toolkit:ListPicker x:Name="lstUpdate" HorizontalAlignment="Left" Margin="10,58,0,0" VerticalAlignment="Top" Width="436"/>
                    <TextBlock x:Name="lbl2" HorizontalAlignment="Left" Margin="10,26,0,0" TextWrapping="Wrap" Text="Item to update" VerticalAlignment="Top"/>
                    <TextBlock x:Name="lbl3" HorizontalAlignment="Left" Margin="10,136,0,0" TextWrapping="Wrap" Text="new name:" VerticalAlignment="Top"/>
                    <TextBox x:Name="txtUpdate" HorizontalAlignment="Left" Height="72" Margin="0,163,0,0" TextWrapping="Wrap" VerticalAlignment="Top" Width="456"/>
                    <Button x:Name="btnUpdate" Content="Update" HorizontalAlignment="Left" Margin="330,235,0,0" VerticalAlignment="Top" Click="btnUpdate_Click"/>
                </Grid>
            </phone:PivotItem>

            <phone:PivotItem Header="Delete">
                <Grid>
                    <toolkit:ListPicker x:Name="lstDelete" HorizontalAlignment="Left" Margin="10,58,0,0" VerticalAlignment="Top" Width="436"/>
                    <TextBlock x:Name="lbl4" HorizontalAlignment="Left" Margin="10,26,0,0" TextWrapping="Wrap" Text="Item to delete" VerticalAlignment="Top"/>
                    <Button x:Name="btnDelete" Content="Delete" HorizontalAlignment="Left" Margin="328,141,0,0" VerticalAlignment="Top" Click="btnDelete_Click"/>
                </Grid>
            </phone:PivotItem>


            <phone:PivotItem Header="View">
                <ListBox x:Name="lstView" HorizontalAlignment="Left" Height="593" VerticalAlignment="Top" Width="456"/>
            </phone:PivotItem>
        </phone:Pivot>
    </Grid>

</phone:PhoneApplicationPage>

```

#### Lets Learn the New Stuff

* Open the MainPage.cs file and add this import at the top of the file.

```cs

        using SQLitePCL;

```

* Make a global variable of type SQLiteConnection called conn. Then create a public function called LoadDatabase. This function will create our database file and our main table. It also sets the variable conn to be used later. The contents of the function should look as below.

```cs

        //create a database
        public void LoadDatabase(){
            //set conn
            conn = new SQLiteConnection("demo.db");

            //SQLite sql statement to create a table named name with 2 fields
            string sql = @"CREATE TABLE IF NOT EXISTS Name(ID INTEGER PRIMARY KEY NOT NULL, Name VARCHAR);";

            //this statement exicutes the sql
            using (var statement = conn.Prepare(sql))
            {
                statement.Step();
            }
        }
      
```

* Lets create an insert function for our database.

```cs 

        //insert
        public void insert(string name)
        {
            //can throw an error
            try
            {
                //exicues insert statement from SQL string
                using (var namestmt = conn.Prepare("INSERT INTO Name (Name) VALUES(?)"))
                {
                    //bind gives a value to a ? in a SQL Query. 
                    //Binds parameters are (value number from 1 to max ?'s, a variable to swap with the ?)
                    namestmt.Bind(1, name);
                    namestmt.Step();
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show("Error");
            }
        }

```

* Create update and delete functions similar to above. Code to help is below.

```cs

        //update
        public void update(string oldName, string newName)
        {
            using(var namestmt = conn.Prepare("UPDATE Name SET Name = ? WHERE Name = ?"))
            {
                namestmt.Bind(1, newName);
                namestmt.Bind(2, oldName);
                namestmt.Step();
            }
        }

        //delete
        public void delete(string name)
        {
            using (var namestmt = conn.Prepare("DELETE FROM Name WHERE Name = ?"))
            {
                namestmt.Bind(1, name);
                namestmt.Step();
            }
        }

```

* We need to create a Select all function, this will be used to set the sources for the listBox and listPickers.

```cs

      //select all
      public List<string> selectAll()
      {
          //list to poplate and return
          List<string> names = new List<string>();
          //exicute sql statement
          using (var statement = conn.Prepare("SELECT * FROM Name"))
          {
              //exicutes while the step(think of it as a result row parser) has rows with values
              while (statement.Step() == SQLiteResult.ROW)
              {
                  //adds the name from the db to the list
                  names.Add((string) statement[1]);
              }
          }
          return names;
      }

```

* Next we need to make a function to set the item source for our boxes and pickers.

```cs

      //set all lists to new data
      public void updateViewsAndControls()
      {
          //create a list and set it to the select all statement
          List<string> list = selectAll();
          //converts the list to an array
          string[] names = list.ToArray();
          //sets the item sources to the array
          lstView.ItemsSource = names;
          lstDelete.ItemsSource = names;
          lstUpdate.ItemsSource = names;
      }

```

* Almost there! Add these function calls to your cunstructor.

```cs 

      LoadDatabase();
      updateViewsAndControls();

```

* Finaly Program the Insert, Update and Delete Buttons to the coresponding functions below. Then run and test the application.

```cs 

      private void btnInsert_Click(object sender, RoutedEventArgs e)
      {
          string name = txtInsert.Text;
          insert(name);
          updateViewsAndControls();
          MessageBox.Show("Success!");
      }

      private void btnUpdate_Click(object sender, RoutedEventArgs e)
      {
          string newName = txtUpdate.Text, oldName = lstUpdate.SelectedItem.ToString();
          update(oldName, newName);
          updateViewsAndControls();
      }

      private void btnDelete_Click(object sender, RoutedEventArgs e)
      {
          string name = lstDelete.SelectedItem.ToString();
          delete(name);
          updateViewsAndControls();
      }

```

[img1]: img/fimg1.png "Tutorial img 1 shows a visual of above text. Created by Caleb Wagner."
[img2]: img/fimg2.png "Tutorial img 2 shows a visual of above text. Created by Caleb Wagner."
[img3]: img/fimg3.png "Tutorial img 3 shows a visual of above text. Created by Caleb Wagner."
[img4]: img/fimg4.png "Tutorial img 4 shows a visual of above text. Created by Caleb Wagner."
[img5]: img/fimg5.png "Tutorial img 5 shows a visual of above text. Created by Caleb Wagner."
[img6]: img/fimg6.png "Tutorial img 6 shows a visual of above text. Created by Caleb Wagner."
[img7]: img/fimg7.png "Tutorial img 7 shows a visual of above text. Created by Caleb Wagner."

