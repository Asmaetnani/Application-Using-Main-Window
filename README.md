# Application-Using-Main-Window  

 A main window provides a framework for building an application's user interface. Qt has QMainWindow and its related classes for main window management. QMainWindow has its own layout to which you can add QToolBars, QDockWidgets, a QMenuBar, and a QStatusBar. The layout has a center area that can be occupied by any kind of widget. You can see an image of the layout below.  
 
 ![image](https://user-images.githubusercontent.com/86807044/146684471-15b13ebd-21e4-440b-9c9d-f59d2f2c68c2.png)
 

# SpreadSheet 

**image**  

Introduction
-
In general a spreadsheet is a file that exists of cells in rows and columns and can help arrange, calculate and sort data. Data in a spreadsheet can be numeric values, as well as text, formulas, references and functions.  
In this project our task is to use QMainWindow to write the code for the graphical and set of actions for our main SpreadSeet application, and then we will code a set of basic functionalities.  
  
The application now looks like this:  

**image**

1-Go cell
-
-The function go cell will give us the location of any cell we're searching.-   

To do so we first need to create a dialog for the user to select a cell.   
Here is the code for the **godialog.h** in which we added a public Getter for the line edit Text to get the cell address.
```cpp
#ifndef GODIALOG_H
#define GODIALOG_H

#include <QDialog>

namespace Ui {
class GoDialog;
}

class GoDialog : public QDialog
{
    Q_OBJECT

public:
    explicit GoDialog(QWidget *parent = nullptr);
    ~GoDialog();
    QString getcell()const; //getter pour le text de lineedit

private:
    Ui::GoDialog *ui;
};

#endif // GODIALOG_H
```
Then, in **godialog.cpp** we added a regular expression validator for the lineEdit.   
```cpp
#include "godialog.h"
#include "ui_godialog.h"
#include <QRegExp>
#include <QRegExpValidator>

GoDialog::GoDialog(QWidget *parent) :
    QDialog(parent),
    ui(new Ui::GoDialog)
{
    ui->setupUi(this);


     QRegExp exp{"[A-Z][1-9][0-9]{0,2}"};

    ui->lineEdit->setValidator(new QRegExpValidator(exp));
}

GoDialog::~GoDialog()
{
    delete ui;
}
QString GoDialog::getcell()const{
    return ui->lineEdit->text();
}

```
And this is the implementation of the final function.
```cpp
void SpreadSheet::goCellSlot(){
    //1 creer le dialogue

    GoDialog D;

    //2 executer le dialogue

   auto reply = D.exec();

    //3 verifier si le dialogue a ete accepte

   if(reply == GoDialog::Accepted)
      {       //extraire le texte

       QString cell = D.getcell();

       //extraire la ligne

       int row = cell[0].toLatin1() -'A';

       //extraire la colonne

       cell = cell.remove(0,1);
       int col = cell.toInt()-1;

       //4changer de cellule
       QStatusBar().showMessage("Changing the current cell",2000);
       spreadsheet->setCurrentCell(row,col);

   }
}
```

**image**

2-Find Dialog
-
We will move now for the Find dialog. This dialog prompts the user for a input and seek a cell that contains the entered text.  
To do so we will use a dialog another time for the user to select the text he's searching for.   

This is the code for the **finddialog.h**.
```cpp
#ifndef FINDDIALOG_H
#define FINDDIALOG_H

#include <QDialog>

namespace Ui {
class findDialog;
}

class findDialog : public QDialog
{
    Q_OBJECT

public:
    explicit findDialog(QWidget *parent = nullptr);
    ~findDialog();
    QString getcelltext()const;

private:
    Ui::findDialog *ui;
};

#endif // FINDDIALOG_H

```
And here the code for the **finddialog.cpp**;
```cpp
#include "finddialog.h"
#include "ui_finddialog.h"

findDialog::findDialog(QWidget *parent) :
    QDialog(parent),
    ui(new Ui::findDialog)
{
    ui->setupUi(this);
}

findDialog::~findDialog()
{
    delete ui;
}
QString findDialog::getcelltext()const{
    return ui->lineEdit->text();
}
```
And finally the implemantation of the whole function.
```cpp
void SpreadSheet::gocelltext(){
    findDialog F ;
    int z=0;

    statusBar()->showMessage("Find dialog",2000);
    auto reply = F.exec();
    if(reply == findDialog::Accepted){
        QString cell =F.getcelltext();
        for(auto i=0;i<=spreadsheet->rowCount()-1;i++){
            if(z>0)
                break;

            for(auto j=0;j<=spreadsheet->columnCount()-1;j++){
                if(spreadsheet->item(i,j) !=NULL){
                    if(spreadsheet->item(i,j)->text() == cell){
                        z++;

                spreadsheet->setCurrentCell( i,  j);}


                }
              }
        }
    }}
```
**image**

3-Saving and loading files in a simple format
-
## Saving files  

To save the content of our spreadsheet in a simple format we will first crete a saveSlot that will check if the file have a name if not it will ask you to give a name to that file.  
He're is the code for it .
```cpp
void SpreadSheet::saveSlot(){
    if(!currentfile){
     QFileDialog D;
    auto filename = D.getSaveFileName();
    currentfile = new QString(filename);
    setWindowTitle(*currentfile);}

//sauvegarder le contenu
 saveContent(*currentfile);

}
```
After we implement the save function using the following code :  
```cpp
void SpreadSheet::saveContent(QString filename) const{

    //1 pointeur sur le fichier d'interet
    QFile file(filename);

    //ouvrir le fichier en mode write

    if(file.open(QIODevice::WriteOnly)){
        QTextStream out(&file);
        //parcourir les cellules et sauvegarder leur contenu
        for(int i=0;i<spreadsheet->rowCount();i++)
            for(int j=0;j<spreadsheet->columnCount();j++)
            {auto cell = spreadsheet->item(i,j);
                if(cell)
                  { out<< i << "," << j << "," <<cell->text() << endl;}
    }

}}
```
**image**

## Loading files  

To open the files we already saved we will have to create a slot to respond to the action trigger in the header .  
This is the code for it: 
```cpp
void SpreadSheet::openSlot(){
    QFileDialog D;
    auto filename = D.getOpenFileName();
    if(filename != ""){
        currentfile = new QString(filename);
        setWindowTitle(filename);
        loadContent(filename);

    }

}

```
And then the function that will open the content of our saved spreadsheet in a simple format with the following code : 
```cpp
void SpreadSheet::loadContent(QString filename){
    QFile file(filename);
    if(file.open(QIODevice::ReadOnly))
    {    //Text Strem
        QTextStream in(&file);
        QString line;
        while(!in.atEnd()){
            line = in.readLine();
            auto tokens = line.split(QChar(','));
            int row = tokens[0].toInt();
            int col = tokens[1].toInt();
            spreadsheet->setItem(row,col,new QTableWidgetItem(tokens[2]));
        }

    }
}
```
**image**  

4-Saving and loading files in a Csv format
-
## Saving files 
We can also save files in a csv format which stores both empty and non empty cells.   
We will first add a slot to respond to the action trigger in the header , and here's the code for that:
```cpp
void SpreadSheet::saveascsvSlot(){
    if(!currentfile){
     QFileDialog D;
    auto filename = D.getSaveFileName();
    currentfile = new QString(filename);
    setWindowTitle(*currentfile);}

//sauvegarder le contenu
 savecsvContent(*currentfile);

}
```
And then the implementation of the function that will save the spreadsheet as a csv format :
```cpp
void SpreadSheet::savecsvContent(QString filename) {
    QFile file(filename);

   int rows = spreadsheet->rowCount();
   int columns = spreadsheet->columnCount();
   if(file.open(QIODevice::WriteOnly )) {

       QTextStream out(&file);


   for (int i = 0; i < rows; i++) {
       for (int j = 0; j < columns; j++) {

           auto cell = spreadsheet->item(i,j);
           if(cell)
               out<< cell->text()<<",";
           else
               out<< " "<<",";

       }
        out<< Qt::endl;

   }


   file.close();
}
}
```
## Loading files
To open the files that we saved in a csv format we will first create a slot to respond to the action trigger in the header , and here's the code for it:
```cpp
void SpreadSheet::OpencsvSlot(){

        QFileDialog d;
        auto filename = d.getOpenFileName();

        if(filemenu)

       {
            currentFile = new QString(filename);
            setWindowTitle(*currentFile);

          LoadContentCsv(*currentFile);

        }

}

```
And then the implementation of the function that will open the spreadsheet in a csv format :
```cpp
void SpreadSheet::LoadContentCsv(QString filename){
         QFile file(filename);
         QStringList list;
         int count = 0;
          if(file.open(QIODevice::ReadOnly))
          {
              QTextStream in(&file);

              while( !in.atEnd())
              {
                  QString line = in.readLine();
                  for(auto content : line.split(","))
                      list.append(content);
                  for(auto i =0; i<list.length(); i++)
                      spreadsheet->setItem(count,i, new QTableWidgetItem(list[i]));
                 count++;
                 list.clear();
              }


          }
         file.close();

}
```













