# Application-Using-Main-Window  

 A main window provides a framework for building an application's user interface. Qt has QMainWindow and its related classes for main window management. QMainWindow has its own layout to which you can add QToolBars, QDockWidgets, a QMenuBar, and a QStatusBar. The layout has a center area that can be occupied by any kind of widget. You can see an image of the layout below.  
 
 ![image](https://user-images.githubusercontent.com/86807044/146684471-15b13ebd-21e4-440b-9c9d-f59d2f2c68c2.png)
 

# SpreadSheet 

![ph](https://user-images.githubusercontent.com/93820154/146686083-45489e8f-d88a-4027-b10a-9d6d3ad4d3bd.jpg)

Introduction
-
In general a spreadsheet is a file that exists of cells in rows and columns and can help arrange, calculate and sort data. Data in a spreadsheet can be numeric values, as well as text, formulas, references and functions.  
In this project our task is to use QMainWindow to write the code for the graphical and set of actions for our main SpreadSeet application, and then we will code a set of basic functionalities.  
  
The application now looks like this:  

![WhatsApp Image 2021-12-19 at 18 31 22](https://user-images.githubusercontent.com/93820154/146686097-47f7da62-0404-41f9-8178-98167fc8419f.jpeg)

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

![WhatsApp Image 2021-12-19 at 18 33 58](https://user-images.githubusercontent.com/93820154/146686107-65714e47-6098-4eb6-9571-5236f29679fe.jpeg)

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
![WhatsApp Image 2021-12-19 at 18 34 42](https://user-images.githubusercontent.com/93820154/146686121-1be2c5ff-f862-407b-b216-853d6772f2d5.jpeg)

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
![WhatsApp Image 2021-12-19 at 18 35 08](https://user-images.githubusercontent.com/93820154/146686135-b121af42-113c-4f62-a1ab-0d2b4f45f35e.jpeg)

This is what the saved file looks like :

![WhatsApp Image 2021-12-19 at 18 42 51](https://user-images.githubusercontent.com/93820154/146686149-37b8b518-67c7-43cb-af63-bdcbc1e151dc.jpeg)

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
![WhatsApp Image 2021-12-19 at 18 43 09](https://user-images.githubusercontent.com/93820154/146686157-e5e51048-891b-4978-b8c6-c30ab66db6c4.jpeg)

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

# Text Editor
Introduction
-
A text editor is a computer program that lets a user enter, change, store, and usually print text (characters and numbers, each encoded by the computer and its input and output devices, arranged to have meaning to users or to other programs). Typically, a text editor provides an "empty" display screen (or "scrollable page") with a fixed-line length and visible line numbers.   

For this second application we will try to use the QtDesigner to create a simple text editor program built around QPlainText.    


![b7e27e2a-794d-40d4-bac9-69f220ee4d18](https://user-images.githubusercontent.com/93820154/146686207-62a05368-4e81-4764-927f-23eced11381c.jpg)

Below is the code for the functionalities of the actions in the menus :
```cpp
void textEditor::on_actionNew_triggered()
{  currentFile.clear();
    currentFile=nullptr;
    ui->plainTextEdit->setPlainText(QString())
}

void textEditor::on_actionOpen_triggered()
{
    QString filename= QFileDialog::getOpenFileName(this,"Open the file");
    QFile file(filename);
    currentFile = filename;
    if(!file.open(QIODevice::ReadOnly | QFile::Text))
        QMessageBox::warning(this,"Warning","Cannot open file :"+file.errorString());
    setWindowTitle(filename);
    QTextStream in(&file);
    QString text = in.readAll();
    ui->plainTextEdit->setPlainText(text);
    file.close();
}

void textEditor::on_action_Copy_triggered()
{
    ui->plainTextEdit->copy();
}


void textEditor::on_action_Paste_triggered()
{
   ui->plainTextEdit->paste();
}


void textEditor::on_action_Cut_triggered()
{
    ui->plainTextEdit->cut();
}

void textEditor::on_actionAbout_Qt_triggered()
{
    QMessageBox::aboutQt(this, "About Me");
}


```
# Conclusion
In this session we enjoyed working with the QtDesigner because it made it easier for use to achieve a lot of complex applications .











