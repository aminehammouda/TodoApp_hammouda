# ToDoApp
# summary   
- [Overview](#Overview)
- [the application interface](#the-application-interface)
- [Defining a Task](#Defining-a-Tasks)
- [Add task function](#Add-task-function)
- [hide/show the pending and finished views](#hide/show-the-pending-and-finished-views)
- [Save tasks](#Save-tasks)
- [Open tasks](#Open-tasks)
- [Change the content of an item](#Change-the-content-of-an-item)
- [MVC model](#MVC-model)
- [THE END](#THE-END)

## Overview
In this project we are aiming to create an application that manage your tasks, of course it should maintain all the features of main application such as menus, actions and toolbar. The pending and finished tasks must be stored in an arshive. 

![1](https://user-images.githubusercontent.com/86841843/150674070-63a90a6c-d587-4596-bd2e-6159537e9738.png)


## the application interface

Using Qt Designer we create the base shape of our App:

![2](https://user-images.githubusercontent.com/86841843/150674083-54e97d9f-e6e5-4ad6-9136-2d4a5eed5c1f.png)

## Defining a Task
Now it s time to make a dialog where we are going to define our tasks

A Task is defined by the following attributes:
- A description: stating the text and goal for the task like (Buying the milk).
- A finished Boolean indicating if the task is Finished or due.
- A Tag category to show the class of the task which is reduced to the following values:
  - Work
  - Life
  - Other
- Finally, a task should have a DueDate which stores the Date planned for the date.

we will use Qt designer to create the dialog

![3](https://user-images.githubusercontent.com/86841843/150674099-ea4aeaf1-a790-44ed-aec2-c447e2fdf245.png)

Now , we will add setters :
- date setter 
```cpp
void addTaskDialog::setdate(int y,int m, int d){
    QDate da;

    da.setDate(y,m,d);
    ui->dateEdit->setDate(da);
}
```
- description setter
```cpp
void addTaskDialog::setdesc(QString a){
    ui->lineEdit->setText(a);
}
```
- tag setter
```cpp
void addTaskDialog::settag(QString a){
    ui->comboBox->setCurrentText(a);
}
```
- checkbox setter
```cpp
void addTaskDialog::setf(bool a){
    ui->checkBox->setChecked(a);
}
```
- gettext function
```cpp
QString addTaskDialog::getText(){

    QString a= ui->lineEdit->text() + " Due : " + ui->dateEdit->text() + "  Tag : " + ui->comboBox->currentText() +"";
    return a;
}
```
The function below will check if the checkbox is checked
```cpp
bool addTaskDialog::isChecked(){
    if(ui->checkBox->isChecked()){
        return true;
    }
    return false;
}
```
A minimum date is also required :
```cpp
    QDate date =QDate::currentDate();
    ui->dateEdit->setMinimumDate(date);
    ui->dateEdit->setDate(date);
```

![4](https://user-images.githubusercontent.com/86841843/150674116-fb0f7584-d02c-49a8-89b5-418ee4ac58ba.png)

## Add task function
In our header , we will add a slot :
```cpp
private slots :
  void on_action_Add_triggered();
```
And now the accomplishment of our function :
```cpp
     //first we call the task dialog
     addTaskDialog dialog;
     auto reply= dialog.exec();
     if(reply == addTaskDialog::Accepted)
     {
         //we save the task on a text string using the gettext function
         QString text= dialog.getText();
         // now based on the date and the checkbox status we will decide on wich listwidget we are going to add our task as an item
         if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
             ui->listWidget->addItem(text);
         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
             ui->lw2->addItem(text);
         }else if(dialog.isChecked()){
             ui->lw3->addItem(text);
         }
     }
```
## hide/show the pending and finished views
The user can either hide/show the pending and finished views.
For this we need to add the following slots in our header:
```cpp
private slots:
    void on_action_View_pending_tasks_toggled(bool arg1);
    void on_action_View_finished_tasks_toggled(bool arg1);
```
- the implementation of on_action_View_pending_tasks_toggled slot
```cpp
void ToDoApp::on_action_View_pending_tasks_toggled(bool arg1)
{
    if(arg1){
        ui->lw2->setVisible(true);
    }else{
        ui->lw2->setVisible(false);
    }
}
```
- the implementation of on_action_View_finished_tasks_toggled slot
```cpp
void ToDoApp::on_action_View_finished_tasks_toggled(bool arg1)
{
    if(arg1){
        ui->lw3->setVisible(true);
    }else{
        ui->lw3->setVisible(false);
    }
}
```
To make sure that these widgets are invisible by default we have to add this tow lines to the constructor 
```cpp
   ui->lw2->setVisible(false);
   ui->lw3->setVisible(false);
```
## Save tasks
By to override a close event we will be able to create a save function that enable us to save our tasks after closing our application :

```cpp
protected:
    void closeEvent(QCloseEvent* e) override;
```
The tasks will be saved on a .txt file :
```cpp
void ToDoApp::closeEvent(QCloseEvent* e){
    
    QFile file("C:/Users/hammouda/C++/save.txt");
    if(file.open(QIODevice::ReadWrite | QIODevice::Text)){
        QTextStream out(&file);
        
        for(int i=0;i<ui->listWidget->count();i++)
        {
            out << "1"<< ui->listWidget->item(i)->text() << Qt::endl;
        }
        for(int i=0;i<ui->lw2->count();i++)
        {
            out << "2"<< ui->lw2->item(i)->text() << Qt::endl;
        }
        for(int i=0;i<ui->lw3->count();i++)
        {
            out << "3"<< ui->lw3->item(i)->text() << Qt::endl;
        }
        file.close();
    }
}
```
## Open tasks
We need to add this code to the constructor so that the tasks entered to our application will remains there for future use
to do this 
```cpp
      QFile file("C:/Users/hammouda/C++/save.txt");

      if (!file.open(QIODevice::ReadOnly | QIODevice::Text))
            return;
      while (!file.atEnd()) {
          QString line = file.readLine();
          if(line.at(0)=="1"){
                ui->listWidget->addItem(line.mid(1,line.size()));
          }else if(line.at(0)=="2"){
              ui->lw2->addItem(line.mid(1,line.size()));
          }else if(line.at(0)=="3"){
              ui->lw3   ->addItem(line.mid(1,line.size()));
          }
      }
```
## Change the content of an item
First we have to link the list widget to a function that is automatically executed if an item is double clicked :
```cpp
connect(ui->listWidget, SIGNAL(itemDoubleClicked(QListWidgetItem*)), this,SLOT(sss()));
connect(ui->lw2, SIGNAL(itemDoubleClicked(QListWidgetItem*)), this,SLOT(sss2()));
connect(ui->lw3, SIGNAL(itemDoubleClicked(QListWidgetItem*)), this,SLOT(sss3()));
```
We have to add these three slots to the header :
```cpp
private slots:    
    void sss();
    void sss2();
    void sss3();
```
now the employment of the slots sss():
```cpp
void ToDoApp::sss(){
    addTaskDialog dialog;
    int i=1;
    
    QString task;
    // declare the new task
    QListWidgetItem *a=ui->listWidget->currentItem();
    // now we split the task 
    QStringList list = a->text().split(QRegularExpression("\\W+"), Qt::SkipEmptyParts);
    task+=list[0];
    
    while(list[i]!="Due"){
         task+=" "+list[i];
         i++;
    }
    // set the date
   dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    // set the description
    dialog.setdesc(task);
    //set the tag
    dialog.settag(list[i+5]);
    auto reply= dialog.exec();
    if(reply == addTaskDialog::Accepted)
     {
         QString text= dialog.getText();
         if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
             ui->listWidget->addItem(text);
         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
             ui->lw2->addItem(text);
         }else if(dialog.isChecked()){
             ui->lw3->addItem(text);
         }
         // delete the previous item so we won't have a duplicated one 
         delete a ;
     }
}
```
It s the same for the second slot :
```cpp
void ToDoApp::sss(){
    addTaskDialog dialog;
    int i=1;
    
    QString task;
    // declare the new task
    QListWidgetItem *a=ui->listWidget->currentItem();
    // now we split the task 
    QStringList list = a->text().split(QRegularExpression("\\W+"), Qt::SkipEmptyParts);
    task+=list[0];
    
    while(list[i]!="Due"){
         task+=" "+list[i];
         i++;
    }
    
    // set the date
    dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    // set the description
    dialog.setdesc(task);
    //set the check box for finished tasks to true
    dialog.setf(true);
    //set the tag
    dialog.settag(list[i+5]);
    auto reply= dialog.exec();
    if(reply == addTaskDialog::Accepted)
     {
         QString text= dialog.getText();
         if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
             ui->listWidget->addItem(text);
         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
             ui->lw2->addItem(text);
         }else if(dialog.isChecked()){
             ui->lw3->addItem(text);
         }
         // delete the previous item so we won't have a duplicated one 
         delete a ;
     }
}
```
Lets  test it out :
- step 1: create a task
![6](https://user-images.githubusercontent.com/86841843/150674745-8c4b4a7b-6d9b-44fc-915c-d21b47d6177f.png)

- step 2: double click on the task
![1](https://user-images.githubusercontent.com/86841843/150674334-ac82f818-ebf9-4020-8505-601e4e49d6e9.png)

 - step 3: change the attributes
![66](https://user-images.githubusercontent.com/86841843/150674700-7797e6e0-fe1e-4239-bb9f-f5b4dc4b421f.png)


- step 4: save the changes
![666](https://user-images.githubusercontent.com/86841843/150674797-a8ce4bd4-190b-48f5-8ecc-3b2db8db50ec.png)


## MVC model

now lets make another version of the ToDoApp but now using models

- with the qt designer and using QListView we will create a form :
![fresh](https://user-images.githubusercontent.com/86841843/150674864-08020070-0cf1-4443-838b-5c89015d8fa9.png)

- Using QStringModel will help us to create models : 
  - first let us add this code to the header
 ```cpp
    QStringListModel *model1;
    QStringListModel *model2;
    QStringListModel *model3;
    
    QStringList Todaytasks;
    QStringList Finishedtasks;
    QStringList Pendingtasks;
 ```
 ```cpp
    model1 = new QStringListModel();
    model2 = new QStringListModel();
    model3 = new QStringListModel();
    ```
  - then we have to set the model to the view
  ```cpp
     model1->setStringList(Todaytasks);
     ui->lw1->setModel(model1);


     model2->setStringList(Pendingtasks);
     ui->lw2->setModel(model2);


     model3->setStringList(Finishedtasks);
     ui->lw3->setModel(model3);
  ```
 - the addaction function
 ```cpp
 void ToDoApp::on_action_Add_triggered()
 {
     addTaskDialog dialog;
     auto reply= dialog.exec();
     if(reply == addTaskDialog::Accepted)
     {
         QString text= dialog.getText();
         if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
             Todaytasks.append(text);
         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
             Pendingtasks.append(text);
         }else if(dialog.isChecked()){
             Finishedtasks.append(text);
         }
     }

     model1->setStringList(Todaytasks);
     ui->lw1->setModel(model1);

     model2->setStringList(Pendingtasks);
     ui->lw2->setModel(model2);


     model3->setStringList(Finishedtasks);
     ui->lw3->setModel(model3);
 }
 ```
 - Now the implementation of the new open and save function
   - Save :
  ```cpp
   void ToDoApp::closeEvent(QCloseEvent* e){
    QFile file("C:/Users/zakariae zaoui/Desktop/alo.txt");
    if(file.open(QIODevice::ReadWrite | QIODevice::Text)){
        QTextStream out(&file);
        for(int i=0;i<Todaytasks.size();i++)
        {
            out << "1"<< Todaytasks.at(i)<< Qt::endl;
        }
        for(int i=0;i<Pendingtasks.size();i++)
        {
            out << "2"<< Pendingtasks.at(i) << Qt::endl;
        }
        for(int i=0;i<Finishedtasks.size();i++)
        {
            out << "3"<< Finishedtasks.at(i) << Qt::endl;
        }
        file.close();
    }
   }
   ```
   - open :
     - this will be added to the constructor
   
   ```cpp
      QFile file("C:/Users/hammouda/Desktop/save.txt");

      if (!file.open(QIODevice::ReadOnly | QIODevice::Text))
            return;

      while (!file.atEnd()) {
          QString line = file.readLine();
          if(line.at(0)=="1"){
                Todaytasks.append(line.mid(1,line.size()));
          }else if(line.at(0)=="2"){
              Pendingtasks.append(line.mid(1,line.size()));
          }else if(line.at(0)=="3"){
              Finishedtasks.append(line.mid(1,line.size()));
          }
      }
      
      model1->setStringList(Todaytasks);
      ui->lw1->setModel(model1);

      model2->setStringList(Pendingtasks);
      ui->lw2->setModel(model2);

      model3->setStringList(Finishedtasks);
      ui->lw3->setModel(model3);
   ```
 - Change content of a task
   - first let us connect the lists to the slots that will perform these actions
  ```cpp
      connect(ui->lw1, SIGNAL(doubleClicked(QModelIndex)), this,SLOT(sss()));
      connect(ui->lw2, SIGNAL(doubleClicked(QModelIndex)), this,SLOT(sss2()));
      connect(ui->lw3, SIGNAL(doubleClicked(QModelIndex)), this,SLOT(sss3()));
   ```
   - The implementation of sss() :
   ```cpp
   void ToDoApp::sss(){
    addTaskDialog dialog;
    int i=1;
    QString task;
    QModelIndex index=ui->lw1->currentIndex();
    QString a=Todaytasks.at(index.row());
    QStringList list = a.split(QRegularExpression("\\W+"), Qt::SkipEmptyParts);
    task+=list[0];

    while(list[i]!="Due"){
         task+=" "+list[i];
         i++;
    }

    dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    dialog.setdesc(task);
    dialog.settag(list[i+5]);
    auto reply= dialog.exec();


     if(reply == addTaskDialog::Accepted)
     {
         QString text= dialog.getText();
         if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
             Todaytasks.append(text);
             Todaytasks.removeAt(index.row());

         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
             Pendingtasks.append(text);
             Todaytasks.removeAt(index.row());
         }else if(dialog.isChecked()){
             Finishedtasks.append(text);
             Todaytasks.removeAt(index.row());
         }

     }

     model1->setStringList(Todaytasks);
     ui->lw1->setModel(model1);

     model2->setStringList(Pendingtasks);
     ui->lw2->setModel(model2);


     model3->setStringList(Finishedtasks);
     ui->lw3->setModel(model3);

   } 
   ```
   - The implementation of sss2() :
   ```cpp
   void ToDoApp::sss2 (){
    addTaskDialog dialog;
    int i=1;
    QString task;
    QModelIndex index=ui->lw2->currentIndex();
    QString a=Pendingtasks.at(index.row());
    QStringList list = a.split(QRegularExpression("\\W+"), Qt::SkipEmptyParts);
    task+=list[0];
    while(list[i]!="Due"){
         task+=" "+list[i];
         i++;
    }
    dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    dialog.setdesc(task);
    dialog.settag(list[i+5]);
    auto reply= dialog.exec();
    if(reply == addTaskDialog::Accepted){
         QString text= dialog.getText();
         if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
             Todaytasks.append(text);
             Pendingtasks.removeAt(index.row());
         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
             Pendingtasks.append(text);
   Pendingtasks.removeAt(index.row());
         }else if(dialog.isChecked()){
             Finishedtasks.append(text);
             Pendingtasks.removeAt(index.row());
         }


    }

    model1->setStringList(Todaytasks);
    ui->lw1->setModel(model1);

    model2->setStringList(Pendingtasks);
    ui->lw2->setModel(model2);


    model3->setStringList(Finishedtasks);
    ui->lw3->setModel(model3);
    }
   ```
   - the implementation of sss3()
   ```cpp
    void ToDoApp::sss3(){
    addTaskDialog dialog;
    int i=1;
    QString task;
    QModelIndex index=ui->lw3->currentIndex();
    QString a=Finishedtasks.at(index.row());
    QStringList list = a.split(QRegularExpression("\\W+"), Qt::SkipEmptyParts);
    task+=list[0];
    while(list[i]!="Due"){
         task+=" "+list[i];
         i++;
    }
    dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    dialog.setdesc(task);
    dialog.setf(true);
    dialog.settag(list[i+5]);
    auto reply= dialog.exec();
    if(reply == addTaskDialog::Accepted){
         QString text= dialog.getText();
         if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
             Todaytasks.append(text);
              Finishedtasks.removeAt(index.row());
         }else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
             Pendingtasks.append(text);
              Finishedtasks.removeAt(index.row());
         }else if(dialog.isChecked()){

         }

    }

    model1->setStringList(Todaytasks);
    ui->lw1->setModel(model1);

    model2->setStringList(Pendingtasks);
    ui->lw2->setModel(model2);

    model3->setStringList(Finishedtasks);
    ui->lw3->setModel(model3);
    }
   ```
# Conclusion
working on this project we learned how to crate an app that will help us to manage our time and make us more discipline and that is by organizing our tasks  , also by going across the various stapes of building this project we have learned various tings and developed our problem solving skill . 
