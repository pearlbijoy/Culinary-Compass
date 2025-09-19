#imports
import ast
import sys
import folium
import io
import mysql.connector as mys
from PyQt5.QtCore import *
from PyQt5.QtGui import *
from PyQt5.QtWidgets import *
from PyQt5 import QtWidgets
from PyQt5.uic import loadUi
from PyQt5.QtCore import Qt
from PyQt5.QtWebEngineWidgets import QWebEngineView
from PyQt5.QtWidgets import QVBoxLayout, QMessageBox,QPushButton,QLabel,QScrollArea,QLineEdit

#MAIN SQL CONNECTION
mycon=mys.connect(host='localhost',user='root',passwd='16511812',database='culcompass')
mycursor=mycon.cursor()

u=''   #variable holding the username of the user for the entire code
#wlcm page
class Welcome(QtWidgets.QMainWindow):
    def __init__(self):
        super(Welcome,self).__init__()
        loadUi("wlcm.ui",self)
        self.signupbtn.clicked.connect(self.gotosignuppage)
        self.loginbtn.clicked.connect(self.gotologinpage)
    def gotosignuppage(self):
        widget.setCurrentIndex(1)
    def gotologinpage(self):
        widget.setCurrentIndex(2)
#------------------------------------------------------------------

#signup page
class Signup(QtWidgets.QMainWindow):
    def __init__(self):
        super(Signup,self).__init__()
        loadUi("signup.ui",self)
        self.signup.clicked.connect(self.gotowlcmpage)
        self.signuplogin.clicked.connect(self.gotologinpage)

    def gotowlcmpage(self):
        name=self.lineEditname.text()
        username=self.lineEditusername.text()
        email=self.lineEditemail.text()
        pswd1=self.lineEditpswd1.text()
        pswd2=self.lineEditpswd2.text()
        self.fieldserror.clear()
        self.pswderror.clear()
        #validation,correction stmts        
        if len(username)==0 or len(name)==0 or len(email)==0 or len(pswd1)==0 or len(pswd2)==0:
            self.fieldserror.setText("please input all fields")
        elif not email.endswith('@gmail.com'):
            self.fieldserror.setText('enter valid gmail')
        elif not len(pswd1)>=8:
            self.pswderror.setText("password must be of length 8 or more")
        elif pswd1!=pswd2:
            self.pswderror.setText("Passwords do not Match")

        else:
            exc='select * from userdata where username='+"'"+username+"'"
            mycursor.execute(exc)
            row=mycursor.fetchone()

            if row!=None:
                dialog=QMessageBox(self)
                dialog.setText("username already exists,please try again.")
                dialog.setWindowTitle('Culinary Compass')
                dialog.exec_()

            else:
                mycursor.execute('insert into userdata values(%s,%s,%s,%s)',(username,pswd1,name,email))
                mycon.commit()
                self.lineEditname.clear()
                self.lineEditusername.clear()
                self.lineEditemail.clear()
                self.lineEditpswd1.clear()
                self.lineEditpswd2.clear()

                #user database creation
                myconuser=mys.connect(host='localhost',user='root',passwd='16511812',database="culcompass")
                mycursoruser=myconuser.cursor()
                mycursoruser.execute('create table '+username+'_userdata like culcompass.userdata')
                mycursoruser.execute('create table '+username+'_savedrecipes like culcompass.savedrecipes')
                mycursoruser.execute('create table '+username+'_favrecipes like culcompass.favrecipes')
                mycursoruser.execute('insert into '+username+'_userdata values(%s,%s,%s,%s)',(username,pswd1,name,email))
                myconuser.commit()
                global u
                u=username

                #going to login
                widget.setCurrentIndex(2)
                dialog=QMessageBox(self)
                dialog.setText("Signup succesful! Please login again.")
                dialog.setWindowTitle('Culinary Compass')
                dialog.exec_()

    def gotologinpage(self):
        self.lineEditname.clear()
        self.lineEditusername.clear()
        self.lineEditemail.clear()
        self.lineEditpswd1.clear()
        self.lineEditpswd2.clear()
        widget.setCurrentIndex(2)

#------------------------------------------------------------------        

#login page
class Login(QtWidgets.QMainWindow):
    def __init__(self):
        super(Login,self).__init__()
        loadUi("login.ui",self)
        self.loginbtn.clicked.connect(self.gotohomepage)
        self.loginsignup.clicked.connect(self.gotosignuppage)

    def gotosignuppage(self):
        widget.setCurrentIndex(1)

    def go_to_page(self, page):
        # Navigate to the specified page
        widget.addWidget(page)
        widget.setCurrentWidget(page)

    def gotohomepage(self):
        self.usernameerror.setText('')
        username=self.lineEditusername.text()
        pswd=self.lineEditpswd.text()
        if len(username)==0 or len(pswd)==0:
            self.usernameerror.setText('Please input all fields')
    
        else:
            exc='select * from userdata where username='+"'"+username+"'"+'and pswd='+"'"+pswd+"'"
            mycursor.execute(exc)
            global usersdata
            usersdata=mycursor.fetchone()

            #validation
            if usersdata==None:
                dialog=QMessageBox(self)
                dialog.setText('Invalid username or password. Please try again.')
                dialog.setWindowTitle('Culinary Compass')
                dialog.exec_()     

            else:
                self.lineEditusername.clear()
                self.lineEditpswd.clear()

                #passing info to the homepage
                global u
                u=username
                hom = Homepage()
                usname=username 
                hom.show_name((usname))
                self.go_to_page(hom)

                a="Welcome Back "+usersdata[2]+"!"
                dialog=QMessageBox(self)
                dialog.setText(a)
                dialog.setWindowTitle('Culinary Compass')
                dialog.exec_()
                             
#------------------------------------------------------------------

#homepage
class Homepage(QtWidgets.QMainWindow):
    def __init__(self):

        super(Homepage,self).__init__()
        loadUi("Homepage.ui",self)
        self.stayin.clicked.connect(self.gotostayin)
        self.stepout.clicked.connect(self.gotoexplore1)
        self.about_button.clicked.connect(self.gotoabout)
        self.guidebutton.clicked.connect(self.gotoguide)
        self.accountbtn.clicked.connect(self.gotoacc)
        self.logout.clicked.connect(self.gotowlcmpage)
        self.discover.clicked.connect(self.gotodiscover)
        self.Savedbtn.clicked.connect(self.gotosaved)
        self.fav.clicked.connect(self.gotofav)
       
        mycursor.execute("SELECT quote FROM quotes ORDER BY RAND() LIMIT 1") #selecting random quote
        setquote=mycursor.fetchone()
        self.Quote.setText(setquote[0])
        print(setquote)

    def gotosaved(self):
        print("Going to saved...")
        sav = Saved()
        sav.usersaved()
        self.go_to_page(sav) 

    def gotofav(self):
        print("Going to fav...")
        fav = Favourites()
        fav.userfav()
        self.go_to_page(fav) 

    def show_name(self, uname):
        usname = uname
        self.Nameoftheuser.setText('@'+usname)
        myconuser=mys.connect(host='localhost',user='root',passwd='16511812',database="culcompass")
        mycursoruser=myconuser.cursor()
        srchstmt='select * from '+usname+'_userdata where username='+"'"+str(usname)+"'"
        mycursoruser.execute(srchstmt)
        usr=mycursoruser.fetchone()
        self.rname.setText(usr[2])


    def go_to_page(self, page):
        # Navigate to the specified page
        widget.addWidget(page)
        widget.setCurrentWidget(page)

    def gotoacc(self):
        print("Going to acc...")
        acc = Account()
        global u
        usname=u 
        acc.show_dets((usname))
        self.go_to_page(acc)    
    #page cnnctions
    def gotoabout(self):
        widget.setCurrentIndex(7)
    def gotoguide(self):
        widget.setCurrentIndex(8)
    def gotostayin(self):
        widget.setCurrentIndex(4)
    def gotoexplore1(self):
        widget.setCurrentIndex(5)   
    def gotowlcmpage(self):
        widget.setCurrentIndex(0)
    def gotodiscover(self):
        disc = Discover()
        disc.discfav()
        self.go_to_page(disc)

#----------------------------------------------------------------------------------------------

class RecipeDetailsPage(QtWidgets.QMainWindow):
    def __init__(self):
        super(RecipeDetailsPage, self).__init__()
        loadUi("tryinrecipedetails.ui", self)
        self.back_button = self.findChild(QPushButton, 'back_button')
        self.back_button.clicked.connect(self.goto_previous_page)

        self.recipe_name_label = self.findChild(QLabel, 'recipename')
        self.recipe_details_label = self.findChild(QLabel, 'details')

        self.save_button=self.findChild(QPushButton, 'save')
        self.fav_button=self.findChild(QPushButton,'fav')
        self.fav_button.clicked.connect(self.favouriterecipes)
        
        self.save_button.clicked.connect(self.saverecipes)
        self.homebtn.clicked.connect(self.gotohome)
        self.recipe_name_label.setWordWrap(True)
        self.recipe_details_label.setWordWrap(True)
        
        self.scroll_area_widget = QWidget()
        self.scroll_area_layout = QVBoxLayout(self.scroll_area_widget)
        self.recipe_details_label.setWordWrap(True)

    # Added the label to the layout of the widget
        self.scroll_area_layout.addWidget(self.recipe_details_label)

    # Set the widget containing the label as the scroll area's widget
        self.scrollArea.setWidget(self.scroll_area_widget)
        
    def open_recip_page(self,recipname):
        recipe_name=recipname
        print("Open recipe page called with recipe name:", recipe_name)
        try:
            mycon= mys.connect(host="localhost",user="root",password="16511812",database="culcompass")
            mycursor=mycon.cursor()
            query1 = "SELECT * FROM recipes WHERE name = %s"
            mycursor.execute(query1, (recipe_name,))
            recipe_details = mycursor.fetchall()
            print("Recipe details:", recipe_details)
            
            for row in recipe_details:
                name = row[0]
                minutes = row[1]
                n_steps = row[2]
                steps_str = row[3]
                ingredients_str = row[4]
                n_ingredients = row[5]
                ingredients_str = ingredients_str.strip("[]").replace("'", "").replace('"', '')
                ingredients_str = '"' + ingredients_str + '"'
                ingredients_list = ast.literal_eval(ingredients_str)
                steps_str = steps_str.strip("[]").replace("'", "").replace('"', '')
                steps_str = '"' + steps_str + '"'
                steps_list = ast.literal_eval(steps_str)
                details_text = f"Name: {name}\n \n \n Minutes: {minutes}\n \n \n Number of steps: {n_steps}\n \n \n Steps: {steps_list}\n \n \n Ingredients: {ingredients_list}\n \n \n Number of ingredients: {n_ingredients}"
                details_texts = f" \n Minutes: {minutes}\n \n \n Number of steps: {n_steps}\n \n \n Ingredients: {ingredients_list}\n \n \n  Steps: {steps_list}\n \n \n"
                name_text = f"{name}"
                self.recipe_details_label.setWordWrap(True)
                self.update_recipe_details_label(details_text)
                self.recipe_name_label.setText(name_text.title())

            mycursor.close()
            mycon.close()
        except Exception as e:
            print(f"An error occurred: {e}")

        
    def open_recipe_page(self, recipe_name):
        print("Open recipe page called with recipe name:", recipe_name)
        try:
            mycon= mys.connect(host="localhost",user="root",password="16511812",database="culcompass")
            mycursor=mycon.cursor()
            query1 = "SELECT * FROM recipes WHERE name = %s"
            mycursor.execute(query1, (recipe_name,))
            recipe_details = mycursor.fetchall()
            
            print("Recipe details:", recipe_details)
            
            for row in recipe_details:
                name = row[0]
                minutes = row[1]
                n_steps = row[2]
                steps_str = row[3]
                ingredients_str = row[4]
                n_ingredients = row[5]
                
                ingredients_str = ingredients_str.strip("[]").replace("'", "").replace('"', '')
                ingredients_str = '"' + ingredients_str + '"'
                ingredients_list = ast.literal_eval(ingredients_str)
                steps_str = steps_str.strip("[]").replace("'", "").replace('"', '')
                steps_str = '"' + steps_str + '"'
                steps_list = ast.literal_eval(steps_str)
                details_text = f"Name: {name}\n \n \n Minutes: {minutes}\n \n \n Number of steps: {n_steps}\n \n \n Steps: {steps_list}\n \n \n Ingredients: {ingredients_list}\n \n \n Number of ingredients: {n_ingredients}"
                details_texts = f" \n Minutes: {minutes}\n \n \n Number of steps: {n_steps}\n \n \n Ingredients: {ingredients_list}\n \n \n  Steps: {steps_list}\n \n \n"
                name_text = f"{name}"
                
                self.recipe_details_label.setWordWrap(True)
                
                
                self.update_recipe_details_label(details_text)
                self.recipe_name_label.setText(name_text.title())

            mycursor.close()
            mycon.close()
        except Exception as e:
            print(f"An error occurred: {e}")

    def update_recipe_details_label(self, details_text):
        print("Updating recipe details label with text:", details_text)
        self.recipe_details_label.setText(details_text)
    def go_to_page(self, page):
        # Navigate to the specified page
        widget.addWidget(page)
        widget.setCurrentWidget(page)    
    def gotohome(self):
        print("Going to homepage...")
        global u
        hom = Homepage()
        usname=u 
        hom.show_name((usname))
        self.go_to_page(hom)
    def saverecipes(self):
        mycon= mys.connect(host="localhost",user="root",password="16511812",database="culcompass")
        mycursor=mycon.cursor()
        recipename1 = self.recipe_name_label.text()
        q="select * from "+u+"_savedrecipes where name='"+recipename1+"'"
        mycursor.execute(q)
        mydata=mycursor.fetchone()

        if mydata!=None:
            dialog=QMessageBox(self)
            dialog.setText("Recipe is already saved.")
            dialog.setWindowTitle('Culinary Compass')
            dialog.exec_()
        else:
            try:
                recipename1 = self.recipe_name_label.text()
                recipedetails1=self.recipe_details_label.text()
                print("Recipe name:", recipename1)
                print("Recipe details:", recipedetails1)
                name, minutes, n_steps, steps, ingredients, n_ingredients = self.parse_recipe_details(recipedetails1)
                stepsstr1 = ', '.join(steps)
                ingredientsstr1 = ', '.join(ingredients)

                insert_query = "INSERT INTO "+u+"_savedrecipes (name, minutes, n_steps, steps, ingredients, n_ingredients) VALUES (%s, %s, %s, %s, %s, %s)"
                mycursor.execute(insert_query, (name, minutes, n_steps, stepsstr1, ingredientsstr1, n_ingredients))
                mycon.commit()
                mycursor.close()
                mycon.close()

                QMessageBox.information(self, "Success", "Recipe saved successfully!")
            except Exception as e:
                print(f"An error occurred while saving recipe: {e}")

    def favouriterecipes(self):
        mycon= mys.connect(host="localhost",user="root",password="16511812",database="culcompass")
        mycursor=mycon.cursor()
        recipename1 = self.recipe_name_label.text()
        q="select * from "+u+"_favrecipes where name='"+recipename1+"'"
        mycursor.execute(q)
        mydata=mycursor.fetchone()
        if mydata!=None:
            dialog=QMessageBox(self)
            dialog.setText("Recipe is already added to favourites.")
            dialog.setWindowTitle('Culinary Compass')
            dialog.exec_()
        else:
            try:
                recipename2 = self.recipe_name_label.text()
                recipedetails2=self.recipe_details_label.text()
                name, minutes, n_steps, steps, ingredients, n_ingredients = self.parse_recipe_details(recipedetails2)
                stepsstr2 = ', '.join(steps)
                ingredientsstr2 = ', '.join(ingredients)
                insert_query = "INSERT INTO "+u+"_favrecipes (name, minutes, n_steps, steps, ingredients, n_ingredients) VALUES (%s, %s, %s, %s, %s, %s)"
                mycursor.execute(insert_query, (name, minutes, n_steps, stepsstr2, ingredientsstr2, n_ingredients))
                mycon.commit()

                mycursor.close()
                mycon.close()
                QMessageBox.information(self, "Success", "Recipe added to favourites!")
            except Exception as e:
                print(f"An error occurred while saving recipe: {e}")
            
    def parse_recipe_detailsfav(self,recipedetails2):
        try:
            components2 = recipedetails2.split("\n \n \n")
            name2 = components2[0].split(": ")[1]
            minutes2 = int(components2[1].split(": ")[1])
            n_steps2 = int(components2[2].split(": ")[1])
            steps2 = components2[3].split(": ")[1].split(", ")
            ingredients2 = components2[4].split(": ")[1].split(", ")
            n_ingredients2 = int(components2[5].split(": ")[1])
            print("Parsed Recipe  fav Details:", name2, minutes2, n_steps2, steps2, ingredients2, n_ingredients2)
            return name2, minutes2, n_steps2, steps2, ingredients2, n_ingredients2
        except Exception as e:
            print("Error in parse_recipe_details:", e)
            return None, None, None, None, None, None
    
    def go_to_page(self, page):
        # Navigate to the specified page
        widget.addWidget(page)
        widget.setCurrentWidget(page)    
    def gotohome(self):
        print("Going to homepage...")
        global u
        hom = Homepage()
        usname=u 
        hom.show_name((usname))
        self.go_to_page(hom)
    def parse_recipe_details(self, recipedetails1):
        try:
            components = recipedetails1.split("\n \n \n")
            name = components[0].split(": ")[1]
            minutes = int(components[1].split(": ")[1])
            n_steps = int(components[2].split(": ")[1])
            steps = components[3].split(": ")[1].split(", ")
            ingredients = components[4].split(": ")[1].split(", ")
            n_ingredients = int(components[5].split(": ")[1])
            print("Parsed Recipe Details:", name, minutes, n_steps, steps, ingredients, n_ingredients)
            return name, minutes, n_steps, steps, ingredients, n_ingredients
        except Exception as e:
            print("Error in parse_recipe_details:", e)
            return None, None, None, None, None, None
        
    def goto_previous_page(self):
        print("Going to previous page...")
        widget.setCurrentIndex(4)

#----------------------------------------------------------------------------------------------
        
#stayin
class Stayin(QtWidgets.QMainWindow):
    def __init__(self):
        
        super(Stayin, self).__init__()
        loadUi("recipe_search2.ui", self)
        self.gobck.clicked.connect(self.gotohome)
        self.recipelabel.adjustSize() 
        self.searchrecipes.clicked.connect(self.srch)
        self.searchrecipes_3.clicked.connect(self.srch)
        self.scrollArea = self.findChild(QScrollArea, 'scrollArea')
        self.scrollArea.setWidgetResizable(True) #scroll area to adjust accordingly
        self.scrollArea.setVerticalScrollBarPolicy(Qt.ScrollBarAsNeeded)
        self.recipe_layout = QVBoxLayout()  # Layout to hold recipe buttons
        self.scrollAreaWidgetContents.setLayout(self.recipe_layout)
        
        self.label = self.findChild(QLabel,'recipelabel')
        intial_data=""
        self.update_label_with_data(intial_data)
        
    def srch(self):
        try:
            mycon= mys.connect(host="localhost",user="root",password="16511812",database="culcompass")
            mycursor=mycon.cursor()

            line_edit = self.findChild(QLineEdit,'recipelineEdit_3')
            input_text = line_edit.text()

            line_edit2 = self.findChild(QLineEdit,'recipelineEdit')
            input_text2 = line_edit2.text()
            entered_ingredients = input_text2.split(',')
            

            if input_text and input_text2:
                query="SELECT name from recipes WHERE name like '%"+str(input_text)+"%'"+" and"
                for i in range(0,len(entered_ingredients)):
                    if i==0:
                        query += " ingredients LIKE '%{}%'".format(entered_ingredients[i])
                    else:
                        query += " AND ingredients LIKE '%{}%'".format(entered_ingredients[i])
            elif input_text:
                query="SELECT name from recipes WHERE name like '%"+str(input_text)+"%'"
            elif input_text2:
                query="SELECT name from recipes WHERE"
                for i in range(0,len(entered_ingredients)):
                    if i==0:
                        query += " ingredients LIKE '%{}%'".format(entered_ingredients[i])
                    else:
                        query += " AND ingredients LIKE '%{}%'".format(entered_ingredients[i])
            
            mycursor.execute(query)
            mydata=mycursor.fetchall()

            if mycursor.rowcount!=0:
                for i in reversed(range(self.recipe_layout.count())):
                    widgetToRemove = self.recipe_layout.itemAt(i).widget()
                    self.recipe_layout.removeWidget(widgetToRemove)
                    widgetToRemove.setParent(None)

                for row in mydata:
                    name=row[0]
                    recipe_button = QPushButton(name)
                    recipe_button.setFont(QFont('Century Schoolbook', 14))
                    recipe_button.setStyleSheet("QPushButton::hover"
                             "{"
                             "color : red;"
                             "}") 
                    recipe_button.clicked.connect(lambda checked, n=name: self.open_recipe_page(n))  # Pass name to the function
                    self.recipe_layout.addWidget(recipe_button)
                    
                mycursor.close()
                mycon.close()
            else:
                dialog=QMessageBox(self)
                dialog.setText("Sorry, no recipes found :(\nMaybe recheck spelling\nor clear one of the fields")
                dialog.setWindowTitle('Culinary Compass')
                dialog.exec_()
        except Exception as e:
            print(f"An error occurred: {e}")

    def update_label_with_data(self, data):
        self.label.clear()
        self.label.setWordWrap(True)
        self.label.setText(data)
    def go_to_page(self, page):
        # Navigate to the specified page
        widget.addWidget(page)
        widget.setCurrentWidget(page)    
    def gotohome(self):
        print("Going to homepage...")
        global u
        hom = Homepage()
        usname=u 
        hom.show_name((usname))
        self.go_to_page(hom) 
    def open_recipe_page(self, recipe_name):
        recipe_details_page = RecipeDetailsPage()
        recipe_details_page.open_recipe_page(recipe_name)
        widget.addWidget(recipe_details_page)
        widget.setCurrentWidget(recipe_details_page)

#----------------------------------------------------------------------------------------------

#exploreintro page
class Explore1(QtWidgets.QMainWindow):
    def __init__(self):
        super(Explore1,self).__init__()
        loadUi("explore.ui",self)
        self.Explore1btn.clicked.connect(self.gotoexplore2)
        self.Explore1bk.clicked.connect(self.gotohome)
    def gotoexplore2(self):       
        widget.setCurrentIndex(6)

    def go_to_page(self, page):
        # Navigate to the specified page
        widget.addWidget(page)
        widget.setCurrentWidget(page)   

    def gotohome(self):
        print("Going to homepage...")
        global u
        hom = Homepage()
        usname=u 
        hom.show_name((usname))
        self.go_to_page(hom)

#----------------------------------------------------------------------------------------------        

#search for restauarnts page
class Explore2(QtWidgets.QMainWindow):
    def __init__(self):
        super(Explore2,self).__init__()
        loadUi("editedmapwidget.ui",self)
        self.Backbtnmap.clicked.connect(self.gotoexplore1)
        self.selectcity.setStyleSheet("background-color: rgb(255, 255, 255);")
        self.selectcity.addItems(['NULL','New Delhi','Gurgaon','Noida','Ghaziabad','Faridabad','Pune','Ludhiana',
                                  'Kolkata','Mumbai','Bangalore','Chennai','Secunderabad','Hyderabad','Kochi',
                                  'Jaipur','Ahmedabad','Chandigarh','Panchkula','Goa','Lucknow','Indore','Nashik',
                                  'Guwahati','Amritsar','Kanpur','Allahabad','Aurangabad','Bhopal','Ranchi',
                                  'Vizag','Bhubaneshwar','Coimbatore','Mangalore','Vadodara','Nagpur','Agra',
                                  'Dehradun','Mysore','Puducherry','Surat','Varanasi','Patna','Mohali'],)
        
        self.Cuisineselect.addItems(['NULL','indian','north indian','south indian','chinese','arab',
                           'italian','mexican','japanese','thai','french','spanish','fast food',],)
        
        self.Selectcountry.addItems(['NULL','India','Australia','Brazil','Canada','Indonesia','New Zealand','Philipines','Qatar','Singapore',
                                     'South Africa','Srilanka','UAE','United Kingdom','United States'])
        
        self.Rating.addItems(['NULL','1+','2+','3+','4+'])
        
        self.Pricerange.addItems(['NULL','1','2','3','4'])
        layout = QVBoxLayout()
        self.mapwidget.setLayout(layout)
        latitudeofmap=13.095550542739655
        longitudeofmap=80.26676100085544 #initial coordinates of the map

        m = folium.Map(
        	zoom_start=10,
        	location=(latitudeofmap,longitudeofmap))
        # save map data to data object
        data = io.BytesIO()
        m.save(data, close_file=False)

        webView1 = QWebEngineView()
        webView1.setHtml(data.getvalue().decode())

        layout.addWidget(webView1)

        self.searchmap.clicked.connect(self.mapsearchclicked)
        self.srchrestoname.clicked.connect(self.serchrestoname)

    def serchrestoname(self):
        restoname=self.searchrestoname.text()
        query="select * from restaurants where Restaurant_Name="+'"'+restoname+'"'
        print(query)
        mycursor.execute(query)
        mydata=mycursor.fetchall()
        m = folium.Map(
        	zoom_start=5,
        	location=(20.5937,78.962)
        )
        
        for a,b,c,d,e,f,g,h,i,j,k,l,n,o,p in mydata:
            
            iframe = folium.IFrame('Name of the Restaurant: '+b+'<br>'+'Rating: '+str(l)+'<br>'+'Review: '+o+'<br>'+'Votes: '+str(p)+
                                   '<br>'+'Cuisines available: '+i+'<br>'+'Average Cost For Two: '+str(j)+'<br>'+'Price Range: '+str(k)+'<br>'+'Address: '+e+'<br>'+c)
            tooltip=folium.Tooltip(b+' ,click for more info.',style='width:100px; height:30px; white-space:normal;')
            popup = folium.Popup(iframe, min_width=300, max_width=300)
            folium.Marker(location=[h, g],tooltip=tooltip , popup=popup).add_to(m) 
        layout = self.mapwidget.layout()
        data = io.BytesIO()
        m.save(data, close_file=False)
        webView = QWebEngineView()
        webView.setHtml(data.getvalue().decode())
           
        layout.addWidget(webView)
        layout.removeWidget(layout.itemAt(0).widget())

    def mapsearchclicked(self):
        querydict={}
        q2dict={}
        City=self.selectcity.currentText()
        Cuisines=self.Cuisineselect.currentText()
        Aggregate_rating=self.Rating.currentText()[0]
        Price_range=self.Pricerange.currentText()
        Country_Code=self.Selectcountry.currentText()
        query=''
        if City!='NULL' and (Country_Code!='India' and Country_Code!='NULL'):
            dialog=QMessageBox(self)
            dialog.setText("City to be chosen only if the country is India.")
            dialog.setWindowTitle('Culinary Compass')
            dialog.exec_()
        else:
            if Cuisines!='NULL':
                if City!='NULL':
                    querydict['City']=City
                if Aggregate_rating!='N':
                    q2dict['Aggregate_rating']=Aggregate_rating
                if Price_range!='NULL':
                    querydict['Price_range']=Price_range
                if Country_Code!='NULL':
                    querydict['Country_Code']=Country_Code

                query='select * from restaurants where Cuisines like'+"'%"+Cuisines+"%'"
                for i in querydict:
                    query+=' and '+i+'='+"'"+str(querydict[i])+"'"
                for i in q2dict:
                    query+=' and '+i+'>='+str(q2dict[i])
            else:
                if City!='NULL':
                    querydict['City']=City
                if Aggregate_rating!='N':
                    
                    q2dict['Aggregate_rating']=Aggregate_rating
                if Price_range!='NULL':
                    querydict['Price_range']=Price_range
                if Country_Code!='NULL':
                    querydict['Country_Code']=Country_Code
                
                query='select * from restaurants where '
                count=0
                for i in querydict:
                    query+=i+'='+"'"+str(querydict[i])+"'"
                    count+=1
                    if count!=len(querydict):
                        query+=' and '
                for i in q2dict:
                    query+=' and '+i+'>='+str(q2dict[i])
        if query!='':
            print(Aggregate_rating)
            print(query)
            mycursor.execute(query)
            mydata=mycursor.fetchall()

            m = folium.Map(
            zoom_start=5,
                location=(20.5937,78.9629)
            )
                
            for a,b,c,d,e,f,g,h,i,j,k,l,n,o,p in mydata:
                    
                iframe = folium.IFrame('Name of the Restaurant: '+b+'<br>'+'Rating: '+str(l)+'<br>'+'Review: '+o+'<br>'+'Votes: '+str(p)+
                                '<br>'+'Cuisines available: '+i+'<br>'+'Average Cost For Two: '+str(j)+'<br>'+'Price Range: '+str(k)+'<br>'+'Address: '+e+'<br>'+c)
                tooltip=folium.Tooltip(b+' ,click for more info.',style='width:100px; height:50px; white-space:normal;')
                popup = folium.Popup(iframe, min_width=300, max_width=300)
                folium.Marker(location=[h, g],tooltip=tooltip , popup=popup).add_to(m)

                layout = self.mapwidget.layout()
                data = io.BytesIO()
                m.save(data, close_file=False)
                webView = QWebEngineView()
                webView.setHtml(data.getvalue().decode())
                
                layout.addWidget(webView)
                layout.removeWidget(layout.itemAt(0).widget())      

    def showall(self,l):#this is for the discover pg
        list1=l        
        m = folium.Map(
        	zoom_start=5,
        	location=(20.5937,78.962))

        for i in list1:
            query='select * from restaurants where Restaurant_Name='+'"'+i+'"'
            print(query)
            mycursor.execute(query)
            mydata=mycursor.fetchall()
            for a,b,c,d,e,f,g,h,i,j,k,l,n,o,p in mydata:
            
                iframe = folium.IFrame('Name of the Restaurant: '+b+'<br>'+'Rating: '+str(l)+'<br>'+'Review: '+o+'<br>'+'Votes: '+str(p)+
                                       '<br>'+'Cuisines available: '+i+'<br>'+'Average Cost For Two: '+str(j)+'<br>'+'Price Range: '+str(k)+'<br>'+'Address: '+e+'<br>'+c)
                tooltip=folium.Tooltip(b+' ,click for more info.',style='width:100px; height:30px; white-space:normal;')
                popup = folium.Popup(iframe, min_width=300, max_width=300)
                folium.Marker(location=[h, g],tooltip=tooltip , popup=popup).add_to(m)
                print('added')
        layout = self.mapwidget.layout()
        data = io.BytesIO()
        m.save(data, close_file=False)
        webView = QWebEngineView()
        webView.setHtml(data.getvalue().decode())
           
        layout.addWidget(webView)
        layout.removeWidget(layout.itemAt(0).widget())


    def show_restaurant(self, restaurant_info): #also for the discover pg
        # Extract restaurant information
        restoname = restaurant_info
        query='select * from restaurants where Restaurant_Name='+"'"+restoname+"'"
        print(query)
        mycursor.execute(query)
        mydata=mycursor.fetchall()
        m = folium.Map(
        	zoom_start=5,
        	location=(20.5937,78.962))
        
        for a,b,c,d,e,f,g,h,i,j,k,l,n,o,p in mydata:
            
            iframe = folium.IFrame('Name of the Restaurant: '+b+'<br>'+'Rating: '+str(l)+'<br>'+'Review: '+o+'<br>'+'Votes: '+str(p)+
                                   '<br>'+'Cuisines available: '+i+'<br>'+'Average Cost For Two: '+str(j)+'<br>'+'Price Range: '+str(k)+'<br>'+'Address: '+e+'<br>'+c)
            tooltip=folium.Tooltip(b+' ,click for more info.',style='width:100px; height:30px; white-space:normal;')
            popup = folium.Popup(iframe, min_width=300, max_width=300)
            folium.Marker(location=[h, g],tooltip=tooltip , popup=popup).add_to(m) 
        layout = self.mapwidget.layout()
        data = io.BytesIO()
        m.save(data, close_file=False)
        webView = QWebEngineView()
        webView.setHtml(data.getvalue().decode())
           
        layout.addWidget(webView)
        layout.removeWidget(layout.itemAt(0).widget())
        
    def gotoexplore1(self):
        widget.setCurrentIndex(5)

#----------------------------------------------------------------------------------------------

#about us page
class About(QtWidgets.QMainWindow):
    def __init__(self):
        super(About,self).__init__()
        loadUi("About.ui",self)
        self.Aboutusbkbtn.clicked.connect(self.gotohome)
    def go_to_page(self, page):
        widget.addWidget(page)
        widget.setCurrentWidget(page)    
    def gotohome(self):
        print("Going to homepage...")
        global u
        hom = Homepage()
        usname=u 
        hom.show_name((usname))
        self.go_to_page(hom)
     
#----------------------------------------------------------------------------------------------

#guide page
class Guide(QtWidgets.QMainWindow):
    def __init__(self):
        super(Guide,self).__init__()
        loadUi("guide.ui",self)
        self.explorenowbtn.clicked.connect(self.gotoexplore1)
        self.checkitoutbtn.clicked.connect(self.gotoabout)
        self.cooknow.clicked.connect(self.gotorecipepage)
        self.gotoaccbtn.clicked.connect(self.gotoacc)
        self.backtohome.clicked.connect(self.gotohome)
        self.scrollArea.horizontalScrollBar().setEnabled(False)
    def gotoexplore1(self):
        widget.setCurrentIndex(5)
    def gotorecipepage(self):
        widget.setCurrentIndex(4)
    def gotoabout(self):
        widget.setCurrentIndex(7)
    def gotoacc(self):
        print("Going to acc...")
        acc = Account()
        global u
        usname=u 
        acc.show_dets((usname))
        self.go_to_page(acc)
    def go_to_page(self, page):
        # Navigate to the specified page
        widget.addWidget(page)
        widget.setCurrentWidget(page)    
    def gotohome(self):
        print("Going to homepage...")
        global u
        hom = Homepage()
        usname=u 
        hom.show_name((usname))
        self.go_to_page(hom)
    
#----------------------------------------------------------------------------------------------

#account page
class Account(QtWidgets.QMainWindow):
    def __init__(self):
        super(Account,self).__init__()
        loadUi("account_window.ui",self)
        self.backtohomepage.clicked.connect(self.gotohome)
        self.save.clicked.connect(self.saved)

    def show_dets(self, uname):
        usrname = uname
        myconuser=mys.connect(host='localhost',user='root',passwd='16511812',database="culcompass")
        mycursoruser=myconuser.cursor()
        mycursoruser.execute('select * from '+usrname+'_userdata')
        global usersdata
        usersdata=mycursoruser.fetchone()
        self.name.setText(usersdata[2])
        self.username.setText(usersdata[0])
        self.email.setText(usersdata[3])

    def go_to_page(self, page):
        # Navigate to the specified page
        widget.addWidget(page)
        widget.setCurrentWidget(page)    
    
    def gotohome(self):
        print("Going to homepage...")
        global u
        
        hom = Homepage()
        usname=u 
        hom.show_name((usname))
        self.go_to_page(hom)
    
    def saved(self):
        oldpaswd=self.oldpswd.text()
        pswd1=self.newpswd.text()
        pswd2=self.pswdconfirm.text()
        newname=self.name.text()
        global u
        if len(pswd1)==0 and len(pswd2)==0 and len(oldpaswd)==0:
            myconuser=mys.connect(host='localhost',user='root',passwd='16511812',database="culcompass")
            mycursoruser=myconuser.cursor()
            mycursoruser.execute('select * from '+u+"_userdata where name='"+newname+"'")
            mydata=mycursoruser.fetchone()
            if mydata==None:
                nameupdation='update '+u+'_userdata set name='+"'"+str(newname)+"'"+' where username='+"'"+str(u)+"'"
                nameupdation1='update culcompass.userdata set name='+"'"+str(newname)+"'"+' where username='+"'"+str(u)+"'"
                mycursoruser.execute(nameupdation)
                myconuser.commit()
                mycursoruser.execute(nameupdation1)
                myconuser.commit()
                dialog=QMessageBox(self)
                dialog.setText("Name saved succesfully.")
                dialog.setWindowTitle('Culinary Compass')
                dialog.exec_()
            else:
                pass
        else:
            if not len(pswd1)>=8:
                self.pswderror.setText("Password must be of length 8 or more")
            elif pswd1!=pswd2:
                self.pswderror.setText("Passwords do not Match")
            
            else:
                self.pswderror.clear()
                
                print(u)
                myconuser=mys.connect(host='localhost',user='root',passwd='16511812',database="culcompass")
                mycursoruser=myconuser.cursor()
                mycursoruser.execute('select * from '+u+'_userdata')
                usrstuff=mycursoruser.fetchone()
                if usrstuff[1]==oldpaswd:
                    self.incrrctpswd.clear()
                    newname=self.name.text()
                    pswdupdation='update '+u+'_userdata set pswd='+"'"+str(pswd1)+"'"+' where username='+"'"+str(u)+"'"
                    nameupdation='update '+u+'_userdata set name='+"'"+str(newname)+"'"+' where username='+"'"+str(u)+"'"
                    nameupdation1='update culcompass.userdata set name='+"'"+str(newname)+"'"+' where username='+"'"+str(u)+"'"
                    pswdupdation1='update culcompass.userdata set pswd='+"'"+str(pswd1)+"'"+' where username='+"'"+str(u)+"'"
                    print(pswdupdation)
                    mycursoruser.execute(nameupdation)
                    myconuser.commit()
                    mycursoruser.execute(nameupdation1)
                    myconuser.commit()
                    mycursoruser.execute(pswdupdation)
                    myconuser.commit()
                    mycursoruser.execute(pswdupdation1)
                    myconuser.commit()
                    self.oldpswd.clear()
                    self.newpswd.clear()
                    self.pswdconfirm.clear()
                    dialog=QMessageBox(self)
                    dialog.setText("Details saved succesfully.")
                    dialog.setWindowTitle('Culinary Compass')
                    dialog.exec_()
                else:
                    self.incrrctpswd.setText('Your old password is incorrect.')
        

#----------------------------------------------------------------------------------------------       
#saved page
class Saved(QtWidgets.QMainWindow):
    def __init__(self):
        super(Saved,self).__init__()
        loadUi("saved2.ui",self)
        
        self.recipe_layout = QVBoxLayout()
        self.about_button.clicked.connect(self.gotoabout)
        self.guidebutton.clicked.connect(self.gotoguide)
        self.accountbtn.clicked.connect(self.gotoacc)
        self.logout.clicked.connect(self.gotowlcmpage)
        self.discover.clicked.connect(self.gotodiscover)
        self.homebtn.clicked.connect(self.gotohome)
        self.scrollAreaWidgetContents.setLayout(self.recipe_layout)
        self.deletebutton.clicked.connect(self.displaycheckboxes)
        self.hidden_button = self.findChild(QtWidgets.QPushButton, 'confirmbutton')  # The button to hide
        self.trigger_button = self.findChild(QtWidgets.QPushButton, 'deletebutton')
        saved_recipes = self.retrieve_saved_recipes()
        self.hidden_button.setVisible(False)
        self.hidden_button.clicked.connect(self.deletesavedrecipes)
        self.trigger_button.clicked.connect(self.show_hidden_button)
        self.checkboxes = []
        self.recipe_texts = []
        
        self.load_recipes()

    def load_recipes(self):
        saved_recipes = self.retrieve_saved_recipes()
        
        self.refresh_recipe_buttons()
         

        for i in reversed(range(self.recipe_layout.count())):
                widgetToRemove = self.recipe_layout.itemAt(i).widget()
                self.recipe_layout.removeWidget(widgetToRemove)
                widgetToRemove.setParent(None)# Call the method to retrieve saved recipes
        for recipe_name in saved_recipes:
            button = QtWidgets.QPushButton(recipe_name)
            
            button.clicked.connect(lambda checked, name=recipe_name: self.open_recipe_page(name))
            self.recipe_layout.addWidget(button)

    def show_hidden_button(self):
        self.hidden_button.setVisible(True)
    def usersaved(self):
        try:
            mycon = mys.connect(host="localhost", user="root", password="16511812", database="culcompass")
            mycursor = mycon.cursor()
            query = "SELECT name FROM "+u+"_savedrecipes"
            mycursor.execute(query)
            saved_recipes = [row[0] for row in mycursor.fetchall()]
            mycursor.close()
            mycon.close()
            self.refresh_recipe_buttons()
            
            for i in reversed(range(self.recipe_layout.count())):
                widgetToRemove = self.recipe_layout.itemAt(i).widget()
                self.recipe_layout.removeWidget(widgetToRemove)
                widgetToRemove.setParent(None)# Call the method to retrieve saved recipes
            for recipe_name in saved_recipes:
                button = QtWidgets.QPushButton(recipe_name)
                button.setFont(QFont('Century Schoolbook', 14))
                button.setStyleSheet("QPushButton::hover"
                             "{"
                             "color : red;"
                             "}") 
                button.clicked.connect(lambda checked, name=recipe_name: self.open_recipe_page(name))
                self.recipe_layout.addWidget(button)
        except mys.Error as error:
            print("Failed to retrieve saved recipes from MySQL:", error)
            return []

    def retrieve_saved_recipes(self):
        try:
            mycon = mys.connect(host="localhost", user="root", password="16511812", database="culcompass")
            mycursor = mycon.cursor()
            query = "SELECT name FROM savedrecipes"
            mycursor.execute(query)
            saved_recipes = [row[0] for row in mycursor.fetchall()]
            mycursor.close()
            mycon.close()
            return saved_recipes
        except mys.Error as error:
            print("Failed to retrieve saved recipes from MySQL:", error)
            return []   
 
    def open_recipe_page(self, recipe_name):
        recipe_details_page = RecipeDetailsPage()
        recipe_details_page.open_recipe_page(recipe_name)
        widget.addWidget(recipe_details_page)
        widget.setCurrentWidget(recipe_details_page)
       
    def displaycheckboxes(self):
        print("Display checkboxes method called")
        try:
            print('Display checkboxes')
            print('Display checkboxes')
            self.checkboxes.clear()  # Clear checkboxes at start
            self.recipe_texts.clear() 
            for i in range(self.recipe_layout.count()):
                item = self.recipe_layout.itemAt(i)
                if item is not None:
                    widget = item.widget()
                    if isinstance(widget, QPushButton):
                        checkbox = QCheckBox()
                        h_layout = QHBoxLayout()
                        h_layout.addWidget(widget)
                        h_layout.addWidget(checkbox)
                        h_layout.setAlignment(Qt.AlignLeft)
                        checkbox.setSizePolicy(QSizePolicy.Fixed, QSizePolicy.Fixed)
                        widget.setSizePolicy(QSizePolicy.Expanding, QSizePolicy.Fixed)
                        container = QtWidgets.QWidget()  
                        container.setLayout(h_layout)  # Set layout for the container
                        self.recipe_layout.insertWidget(i, container)
                        self.checkboxes.append(checkbox)
                        self.recipe_texts.append(widget.text())
            print(f"Checkboxes displayed: {self.checkboxes}")
            print(f"Recipe texts: {self.recipe_texts}")            
        except Exception as e:
            print(f"Exception in displaycheckboxes: {e}")
    
    def refresh_recipe_buttons(self):
        for i in reversed(range(self.recipe_layout.count())):
            widget_to_remove = self.recipe_layout.itemAt(i).widget()
            self.recipe_layout.removeWidget(widget_to_remove)
            widget_to_remove.setParent(None)
        self.checkboxes.clear()
        self.recipe_texts.clear()             

    def deletesavedrecipes(self):
        try:
            mycon = mys.connect(host="localhost", user="root", password="16511812", database="culcompass")
            mycursor = mycon.cursor()
            c=0
            for checkbox, button_text in zip(self.checkboxes, self.recipe_texts):
                if checkbox.isChecked():
                    query="DELETE FROM "+u+"_savedrecipes where name=%s"
                    mycursor.execute(query, (button_text,))
                    c=c+1
            mycon.commit()
            print(f"Deleted {c} recipes from MySQL.")
            mycursor.execute("SELECT count(name) from "+u+"_savedrecipes")
            mydata=mycursor.fetchone()
            mycon.commit()
            mycon.close()
            mycursor.close()
            
            self.refresh_recipe_buttons()
            if mydata[0]==0:
                print("No recipes remaining saved. Removing all buttons.")
                for i in reversed(range(self.recipe_layout.count())):
                    widgetToRemove = self.recipe_layout.itemAt(i).widget()
                    self.recipe_layout.removeWidget(widgetToRemove)
                    widgetToRemove.setParent(None)
            else:
                self.usersaved()
            self.hidden_button.setVisible(False)

        except mys.Error as error:
            print("Failed to delete  recipes from MySQL:", error)

    def gotoacc(self):
        print("Going to acc...")
        acc = Account()
        global u
        usname=u 
        acc.show_dets((usname))
        self.go_to_page(acc) 
    def go_to_page(self, page):
        # Navigate to the specified page
        widget.addWidget(page)
        widget.setCurrentWidget(page)    
    def gotohome(self):
        print("Going to homepage...")
        global u
        hom = Homepage()
        usname=u 
        hom.show_name((usname))
        self.go_to_page(hom)
    def gotoabout(self):
        widget.setCurrentIndex(7)
    def gotoguide(self):
        widget.setCurrentIndex(8)   
    def gotowlcmpage(self):
        widget.setCurrentIndex(0)
    def gotodiscover(self):
        disc = Discover()
        disc.discfav()
        self.go_to_page(disc)

#----------------------------------------------------------------------------------------------

#favourites page
class Favourites(QtWidgets.QMainWindow):
    def __init__(self):
        super(Favourites, self).__init__()
        loadUi("fav.ui", self)
        
        self.recipe_layout = QVBoxLayout()
        self.scrollAreaWidgetContents.setLayout(self.recipe_layout)
        
        self.about_button.clicked.connect(self.gotoabout)
        self.guidebutton.clicked.connect(self.gotoguide)
        self.accountbtn.clicked.connect(self.gotoacc)
        self.homebtn.clicked.connect(self.gotohome)
        self.logout.clicked.connect(self.gotowlcmpage)
        self.discover.clicked.connect(self.gotodiscover)
        
        self.deletebutton.clicked.connect(self.displaycheckboxes)
        self.hidden_button = self.findChild(QtWidgets.QPushButton, 'confirmbutton')
        self.trigger_button = self.findChild(QtWidgets.QPushButton, 'deletebutton')
        
        self.hidden_button.setVisible(False)
        self.hidden_button.clicked.connect(self.deletefavrecipes)
        self.trigger_button.clicked.connect(self.show_hidden_button)
        
        self.scroll_area = self.findChild(QtWidgets.QScrollArea, 'scrollArea')
        self.scroll_area.setHorizontalScrollBarPolicy(Qt.ScrollBarAlwaysOn)
        self.scroll_area.setWidgetResizable(True)
        
        self.checkboxes = []
        self.recipe_texts = []

        self.load_recipes()

    def load_recipes(self):
        saved_recipes = self.retrieve_saved_recipes()
        
        self.refresh_recipe_buttons()
        
        for recipe_name in saved_recipes:
            button = QtWidgets.QPushButton(recipe_name)
            button.clicked.connect(lambda checked, name=recipe_name: self.open_recipe_page(name))
            self.recipe_layout.addWidget(button)

    def userfav(self):
        try:
            mycon = mys.connect(host="localhost", user="root", password="16511812", database="culcompass")
            mycursor = mycon.cursor()
            query = "SELECT name FROM "+u+"_favrecipes"
            mycursor.execute(query)
            saved_recipes = [row[0] for row in mycursor.fetchall()]
            mycursor.close()
            mycon.close()
            
            self.refresh_recipe_buttons()
                
            for recipe_name in saved_recipes:
                button = QtWidgets.QPushButton(recipe_name)
                button.setFont(QFont('Century Schoolbook', 14))
                button.setStyleSheet("QPushButton::hover { color : red; }")
                button.clicked.connect(lambda checked, name=recipe_name: self.open_recipe_page(name))
                self.recipe_layout.addWidget(button)

        except mys.Error as error:
            print("Failed to retrieve saved recipes from MySQL:", error)

    def show_hidden_button(self):
        self.hidden_button.setVisible(True)

    def retrieve_saved_recipes(self):
        try:
            mycon = mys.connect(host="localhost", user="root", password="16511812", database="culcompass")
            mycursor = mycon.cursor()
            query = "SELECT name FROM favrecipes"
            mycursor.execute(query)
            saved_recipes = [row[0] for row in mycursor.fetchall()]
            mycursor.close()
            mycon.close()
            return saved_recipes
        except mys.Error as error:
            print("Failed to retrieve saved recipes from MySQL:", error)
            return []

    def displaycheckboxes(self):
        print("Display checkboxes method called")
        self.checkboxes.clear()  # Clear checkboxes at start
        self.recipe_texts.clear() 
        try:
            for i in range(self.recipe_layout.count()):
                item = self.recipe_layout.itemAt(i)
                if item is not None:
                    widget = item.widget()
                    if isinstance(widget, QPushButton):
                        checkbox = QCheckBox()
                        h_layout = QHBoxLayout()
                        h_layout.addWidget(widget)
                        h_layout.addWidget(checkbox)
                        h_layout.setAlignment(Qt.AlignLeft)
                        checkbox.setSizePolicy(QSizePolicy.Fixed, QSizePolicy.Fixed)
                        widget.setSizePolicy(QSizePolicy.Expanding, QSizePolicy.Fixed)
                        container = QtWidgets.QWidget()
                        container.setLayout(h_layout)
                        self.recipe_layout.insertWidget(i, container)
                        self.checkboxes.append(checkbox)
                        self.recipe_texts.append(widget.text())
            print(f"Checkboxes displayed: {self.checkboxes}")
            print(f"Recipe texts: {self.recipe_texts}")

        except Exception as e:
            print(f"Exception in displaycheckboxes: {e}")

    def refresh_recipe_buttons(self):
        for i in reversed(range(self.recipe_layout.count())):
            widget_to_remove = self.recipe_layout.itemAt(i).widget()
            self.recipe_layout.removeWidget(widget_to_remove)
            widget_to_remove.setParent(None)
        self.checkboxes.clear()
        self.recipe_texts.clear()
        
    def deletefavrecipes(self):
        try:
            mycon = mys.connect(host="localhost", user="root", password="16511812", database="culcompass")
            mycursor = mycon.cursor()
            c = 0
            
            for checkbox, button_text in zip(self.checkboxes, self.recipe_texts):
                if checkbox.isChecked():
                    query = "DELETE FROM "+u+"_favrecipes WHERE name=%s"
                    mycursor.execute(query, (button_text,))
                    c += 1
            mycon.commit()
            print(f"Deleted {c} recipes from MySQL.")
            mycursor.execute("SELECT COUNT(name) FROM "+u+"_favrecipes")
            mydata = mycursor.fetchone()
            mycon.commit()
            mycon.close()
            mycursor.close()
            
            self.refresh_recipe_buttons()
            
            if mydata[0] == 0:
                print("No recipes remaining. Removing all buttons.")
            else:
                self.userfav()
            
            self.hidden_button.setVisible(False)

        except mys.Error as error:
            print("Failed to delete recipes from MySQL:", error)

    def open_recipe_page(self, recipe_name):
        recipe_details_page = RecipeDetailsPage()
        recipe_details_page.open_recipe_page(recipe_name)
        widget.addWidget(recipe_details_page)
        widget.setCurrentWidget(recipe_details_page)
    def gotoacc(self):
        print("Going to acc...")
        acc = Account()
        global u
        usname=u 
        acc.show_dets((usname))
        self.go_to_page(acc) 
    def go_to_page(self, page):
        # Navigate to the specified page
        widget.addWidget(page)
        widget.setCurrentWidget(page)    
    def gotohome(self):
        print("Going to homepage...")
        global u
        hom = Homepage()
        usname=u 
        hom.show_name((usname))
        self.go_to_page(hom)
    def gotoabout(self):
        widget.setCurrentIndex(7)
    def gotoguide(self):
        widget.setCurrentIndex(8)   
    def gotowlcmpage(self):
        widget.setCurrentIndex(0)
    def gotodiscover(self):
        disc = Discover()
        disc.discfav()
        self.go_to_page(disc)
#----------------------------------------------------------------------------------------------

#discoverpg 
class Discover(QtWidgets.QMainWindow):
    def __init__(self):
        super(Discover,self).__init__()
        loadUi("discoverpageedit2.ui",self)
        self.cuisinesview_2.clicked.connect(self.gotomap)
        self.cuisinesview.clicked.connect(self.gotocuisines)
        self.bckbtn.clicked.connect(self.gotohome)

        mycursor.execute("SELECT quote FROM quotes ORDER BY RAND() LIMIT 1")
        setquote=mycursor.fetchone()
        self.Quote.setText(setquote[0])

        #new recipes suggestions

        mycursor.execute("SELECT name,minutes,n_ingredients FROM recipes where length(name)<30 ORDER BY RAND() LIMIT 4")
        suggestrecipe=mycursor.fetchone()
        self.r1.setText(suggestrecipe[0].title())
        self.det1.setText('time:'+str(suggestrecipe[1])+'mins')
        self.s1.setText('no. of ingredients:'+str(suggestrecipe[2]))
        self.r1.clicked.connect(self.showrec)

        suggestrecipe=mycursor.fetchone()
        self.r2.setText(suggestrecipe[0].title())
        self.det2.setText('time:'+str(suggestrecipe[1])+'mins')
        self.s2.setText('no. of ingredients:'+str(suggestrecipe[2]))
        self.r2.clicked.connect(self.showrec)
        
        suggestrecipe=mycursor.fetchone()
        self.r3.setText(suggestrecipe[0].title())
        self.det3.setText('time:'+str(suggestrecipe[1])+'mins')
        self.s3.setText('no. of ingredients:'+str(suggestrecipe[2]))
        self.r3.clicked.connect(self.showrec)

        suggestrecipe=mycursor.fetchone()
        self.r4.setText(suggestrecipe[0].title())
        self.det4.setText('time:'+str(suggestrecipe[1])+'mins')
        self.s4.setText('no. of ingredients:'+str(suggestrecipe[2]))
        self.r4.clicked.connect(self.showrec)
        self.cuisinesview_3.clicked.connect(self.gotostayin)

        #new restos suggestions

        mycursor.execute("SELECT restaurant_name,cuisines,city,country_code FROM restaurants where length(restaurant_name)<30 ORDER BY RAND() LIMIT 8")
        place=mycursor.fetchone()
        self.t1.setText(place[0].title())
        self.u1.setText(place[1])
        self.v1.setText(place[2]+','+place[3])
        self.t1.clicked.connect(self.srchplace1)
        

        place=mycursor.fetchone()
        self.t2.setText(place[0].title())
        self.u2.setText(place[1])
        self.v2.setText(place[2]+','+place[3])
        self.t2.clicked.connect(self.srchplace1)

        place=mycursor.fetchone()
        self.t3.setText(place[0].title())
        self.u3.setText(place[1])
        self.v3.setText(place[2]+','+place[3])
        self.t3.clicked.connect(self.srchplace1)

        place=mycursor.fetchone()
        self.t4.setText(place[0].title())
        self.u4.setText(place[1])
        self.v4.setText(place[2]+','+place[3])
        self.t4.clicked.connect(self.srchplace1)

        place=mycursor.fetchone()
        self.t5.setText(place[0].title())
        self.u5.setText(place[1])
        self.v5.setText(place[2]+','+place[3])
        self.t5.clicked.connect(self.srchplace1)

        place=mycursor.fetchone()
        self.t6.setText(place[0].title())
        self.u6.setText(place[1])
        self.v6.setText(place[2]+','+place[3])
        self.t6.clicked.connect(self.srchplace1)

        place=mycursor.fetchone()
        self.t7.setText(place[0].title())
        self.u7.setText(place[1])
        self.v7.setText(place[2]+','+place[3])
        self.t7.clicked.connect(self.srchplace1)

        place=mycursor.fetchone()
        self.t8.setText(place[0].title())
        self.u8.setText(place[1])
        self.v8.setText(place[2]+','+place[3])
        self.t8.clicked.connect(self.srchplace1)

        #fav recipes 
        mycursor.execute("SELECT name,minutes,n_ingredients FROM favrecipes ORDER BY RAND() LIMIT 3")
        suggestrecipe=mycursor.fetchone()
        self.x1.setText(suggestrecipe[0].title())
        self.x1.clicked.connect(self.showrec)
        self.y1.setText('time:'+str(suggestrecipe[1])+'mins')
        self.z1.setText('no. of ingredients:'+str(suggestrecipe[2]))

        suggestrecipe=mycursor.fetchone()
        self.x2.setText(suggestrecipe[0].title())
        self.y2.setText('time:'+str(suggestrecipe[1])+'mins')
        self.z2.setText('no. of ingredients:'+str(suggestrecipe[2]))
        self.x2.clicked.connect(self.showrec)

        suggestrecipe=mycursor.fetchone()
        self.x3.setText(suggestrecipe[0].title())
        self.y3.setText('time:'+str(suggestrecipe[1])+'mins')
        self.z3.setText('no. of ingredients:'+str(suggestrecipe[2]))
        self.x3.clicked.connect(self.showrec)

        self.gotofavs.clicked.connect(self.gotofav)
            

    def discfav(self):
        mycon = mys.connect(host="localhost", user="root", password="16511812", database="culcompass")
        mycursor = mycon.cursor()
        mycursor.execute("SELECT name, minutes, n_ingredients FROM " + u + "_favrecipes ORDER BY RAND() LIMIT 3")
        recipes = mycursor.fetchall()  # Fetch all the results
        mycon.close()
         
        if not recipes:  # If no records are returned
            self.x1.setText("Please favourite more recipes so")
            self.y1.setText( "that they can be displayed here!")
            self.z1.setText("")
            self.x1.setEnabled(False)

            self.x2.setText("Please favourite more recipes so")
            self.y2.setText("that they can be displayed here!")
            self.z2.setText("")
            self.x2.setEnabled(False)

            self.x3.setText("Please favourite more recipes so")
            self.y3.setText("that they can be displayed here!")
            self.z3.setText("")
            self.x3.setEnabled(False)
        else:  # If there are records, display them
            # For the first square
            suggestrecipe = recipes[0]
            self.x1.setText(suggestrecipe[0].title())
            self.y1.setText("time: " + str(suggestrecipe[1]) + " mins")
            self.z1.setText("no. of ingredients: " + str(suggestrecipe[2]))
            self.x1.clicked.connect(self.showrec)

            # For the second square (if available)
            if len(recipes) > 1:
                suggestrecipe = recipes[1]
                self.x2.setText(suggestrecipe[0].title())
                self.y2.setText("time: " + str(suggestrecipe[1]) + " mins")
                self.z2.setText("no. of ingredients: " + str(suggestrecipe[2]))
                self.x2.clicked.connect(self.showrec)
            else:
                self.x2.setText("Please favourite more recipes so")
                self.y2.setText("that they can be displayed here!")
                self.z2.setText("")
                self.x2.setEnabled(False)

                self.x3.setText("Please favourite more recipes so")
                self.y3.setText("that they can be displayed here!")
                self.z3.setText("")
                self.x3.setEnabled(False)

            # For the third square (if available)
            if len(recipes) > 2:
                suggestrecipe = recipes[2]
                self.x3.setText(suggestrecipe[0].title())
                self.y3.setText("time: " + str(suggestrecipe[1]) + " mins")
                self.z3.setText("no. of ingredients: " + str(suggestrecipe[2]))
                self.x3.clicked.connect(self.showrec)
            else:
                self.x3.setText("Please favourite more recipes so")
                self.y3.setText("that they can be displayed here!")
                self.z3.setText("")
                self.x3.setEnabled(False)

        self.gotofavs.clicked.connect(self.gotofav) 
                   
    def gotostayin(self):
        widget.setCurrentIndex(4)
    
    def showrec(self):
        sender_button = self.sender()
        recipname = sender_button.text()
    
        recipe_name = RecipeDetailsPage()
        recipe_name.open_recip_page((recipname))
        self.go_to_page(recipe_name)

    def srchplace1(self):
        sender_button = self.sender()
        restoname = sender_button.text()
    
        explore_page = Explore2()
        explore_page.show_restaurant((restoname))
        self.go_to_page(explore_page)

    def go_to_page(self, page):
        # Navigate to the specified page
        widget.addWidget(page)
        widget.setCurrentWidget(page)

    def gotomap(self):
        widget.setCurrentIndex(6)
        l=[self.t1.text(),self.t2.text(),self.t3.text(),self.t4.text(),self.t5.text(),self.t6.text(),self.t7.text(),self.t8.text()]
        explore_page = Explore2()
        explore_page.showall((l))
        self.go_to_page(explore_page)
        
    def gotohome(self):
        print("Going to homepage...")
        global u

        hom = Homepage()
        usname=u 
        hom.show_name((usname))
        self.go_to_page(hom)
    
    def gotofav(self):
        print("Going to fav...")
        fav = Favourites()
        fav.userfav()
        self.go_to_page(fav)
 
    def gotocuisines(self):
        widget.setCurrentIndex(12)

#----------------------------------------------------------------------------------------------

#cuisines describe pages
class cusdesc(QWidget):
    def __init__(self, parent = None):
        super(cusdesc, self).__init__(parent)
        loadUi("desc.ui",self)
        self.show()

#----------------------------------------------------------------------------------------------

#cuisines1
class Cuisines1(QtWidgets.QMainWindow):
    def __init__(self, parent=None):
        super(Cuisines1, self).__init__(parent)
        loadUi("discriptionpg1.ui",self)
         
        self.arabian.clicked.connect(self.opendesc)
        self.chinese.clicked.connect(self.opendesc)
        self.french.clicked.connect(self.opendesc)
        self.italian.clicked.connect(self.opendesc)
        self.nextpg.clicked.connect(self.nextpage)
        self.bckbtn.clicked.connect(self.bck)
        self.acceptDrops()

        self.x = 200
        self.y = 200        
    def bck(self):
        widget.setCurrentIndex(10)
    def nextpage(self):
        widget.setCurrentIndex(13)
    def opendesc(self):
        self.opener = cusdesc()
        sender_button = self.sender()
        button_name = sender_button.objectName()
        filename=button_name+'.png'
        print(filename)
        pixmap=QPixmap(filename)
        self.opener.bgpic.setPixmap(pixmap)
        self.opener.show()

#----------------------------------------------------------------------------------------------

class Cuisines2(QtWidgets.QMainWindow):
    def __init__(self):
        super(Cuisines2,self).__init__()
        loadUi("discriptionpg2.ui",self)
        self.thai.clicked.connect(self.opendesc)
        self.si.clicked.connect(self.opendesc)
        self.ni.clicked.connect(self.opendesc)
        self.mexican.clicked.connect(self.opendesc)
        self.japanese.clicked.connect(self.opendesc)
        self.american.clicked.connect(self.opendesc)
        self.bckbtn.clicked.connect(self.bck)
    def opendesc(self):
        self.opener = cusdesc()
        sender_button = self.sender()
        button_name = sender_button.objectName()
        filename=button_name+'.png'
        print(filename)
        pixmap=QPixmap(filename)
        self.opener.bgpic.setPixmap(pixmap)
        self.opener.show()
    def bck(self):
        widget.setCurrentIndex(12)

#----------------------------------------------------------------------------------------------

#concluding statements
app=QtWidgets.QApplication(sys.argv)
widget=QtWidgets.QStackedWidget()

screen00=Welcome()
screen01=Signup()
screen02=Login()
screen03=Homepage()
screen04=Stayin()
screen05=Explore1()
screen06=Explore2()
screen07=About()
screen08=Guide()
screen09=Account()
screen10=Discover()
screen11=Saved()
screen12=Cuisines1()
screen13=Cuisines2()
screen14=Favourites()

#conencting all pages
widget.addWidget(screen00)
widget.addWidget(screen01)
widget.addWidget(screen02)
widget.addWidget(screen03)
widget.addWidget(screen04)
widget.addWidget(screen05)
widget.addWidget(screen06)
widget.addWidget(screen07)
widget.addWidget(screen08)
widget.addWidget(screen09)
widget.addWidget(screen10)
widget.addWidget(screen11)
widget.addWidget(screen12)
widget.addWidget(screen13)
widget.addWidget(screen14)
widget.setWindowTitle('Culinary Compass')
widget.setFixedSize(1750, 900)  
widget.setWindowFlags(Qt.Window | Qt.WindowTitleHint | Qt.WindowCloseButtonHint)
widget.move(60, 65)
widget.show()

sys.exit(app.exec_())