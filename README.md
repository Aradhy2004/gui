import sys
import mysql.connector
mydb=mysql.connector.connect(host='localhost',user='root',passwd='root',database='reportcard')
mycursor=mydb.cursor()

from PyQt6.QtWidgets import (QApplication, QMainWindow, QTabWidget, QWidget, QVBoxLayout, QPushButton, QLabel, QLineEdit, QFormLayout, QHeaderView, QTableWidget, QTableWidgetItem, QDialog, QSizePolicy, QFrame)
from PyQt6.QtCore import Qt
from PyQt6.QtGui import QFont



class AddStudentRecordWindow(QDialog):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Add Student Record")

        layout = QVBoxLayout()
        self.setLayout(layout)

        admno_label = QLabel("Admission No:")
        self.admno_input = QLineEdit()

        name_label = QLabel("Name:")
        self.name_input = QLineEdit()

        fname_label = QLabel("Father's Name:")
        self.fname_input = QLineEdit()

        mname_label = QLabel("Mother's Name:")
        self.mname_input = QLineEdit()

        mno_label = QLabel("Mobile Number:")
        self.mno_input = QLineEdit()

        address_label = QLabel("Address:")
        self.address_input = QLineEdit()

        cat_label = QLabel("Category (general/SC/ST/OBC/other):")
        self.cat_input = QLineEdit()

        add_button = QPushButton("Add Record")
        add_button.clicked.connect(self.add_record)

        layout.addWidget(admno_label)
        layout.addWidget(self.admno_input)
        layout.addWidget(name_label)
        layout.addWidget(self.name_input)
        layout.addWidget(fname_label)
        layout.addWidget(self.fname_input)
        layout.addWidget(mname_label)
        layout.addWidget(self.mname_input)
        layout.addWidget(mno_label)
        layout.addWidget(self.mno_input)
        layout.addWidget(address_label)
        layout.addWidget(self.address_input)
        layout.addWidget(cat_label)
        layout.addWidget(self.cat_input)
        layout.addWidget(add_button)

    def add_record(self):
        try:
            admno = int(self.admno_input.text())
            name = self.name_input.text()
            fname = self.fname_input.text()
            mname = self.mname_input.text()
            mno = int(self.mno_input.text())
            address = self.address_input.text()
            cat = self.cat_input.text()

            dat = (admno, name, fname, mname, mno, address, cat)
            mycursor.execute('INSERT INTO student_info VALUES (%s, %s, %s, %s, %s, %s, %s)', dat)
            mydb.commit()
            print('Record Added')
        except Exception as e:
            print('Problem in adding student data:', e)

class DisplayStudentRecordWindow(QDialog):
    def __init__(self, data):
        super().__init__()
        self.setWindowTitle("Student Information Table")
        self.setGeometry(100, 100, 800, 500)
        layout = QVBoxLayout()
        self.setLayout(layout)

        table = QTableWidget()
        table.setColumnCount(7)  # Number of columns in the table
        table.setRowCount(len(data))  # Number of rows in the table
        table.setHorizontalHeaderLabels(["Admno", "Name", "Father's Name", "Mother's Name", "Mobile No", "Address", "Category"])

        for row, rec in enumerate(data):
            for col, value in enumerate(rec):
                item = QTableWidgetItem(str(value))
                table.setItem(row, col, item)

        layout.addWidget(table)

    def display_info_table(self):
        try:
            query = 'SELECT * FROM student_info'
            mycursor.execute(query)
            myrecords = mycursor.fetchall()

            info_window = DisplayStudentRecordWindow(myrecords)
            info_window.exec()
        except Exception as e:
            print('Something went wrong:', e)

class ReportCardWindow(QDialog):
    def __init__(self, rec):
        super().__init__()
        self.setWindowTitle("Report Card")
        self.setGeometry(100, 100, 500, 500)

        layout = QVBoxLayout()
        self.setLayout(layout)

        university_label = QLabel("<b>SRM University</b>")
        report_card_label = QLabel("<b>Report Card</b>")
        university_label.setAlignment(Qt.AlignmentFlag.AlignCenter)
        report_card_label.setAlignment(Qt.AlignmentFlag.AlignCenter)

        enrollment_label = QLabel(f"<b>Enrollment no.:</b> {rec[0]}")
        name_label = QLabel(f"<b>Name:</b> {rec[1]}")
        course_label = QLabel(f"<b>Course:</b> {rec[2]}")

        separator_frame = QFrame()
        separator_frame.setFrameShape(QFrame.Shape.HLine)
        separator_frame.setFrameShadow(QFrame.Shadow.Sunken)

        table = QTableWidget()
        table.setColumnCount(3)  # Adding a column for grades
        table.setRowCount(len(rec) - 7)  # Excluding Total Marks, Percentage, and Grade
        table.setHorizontalHeaderLabels(["Subject", "Marks", "Grade"])
        table.horizontalHeader().setSectionResizeMode(QHeaderView.ResizeMode.Stretch)
        #table.verticalHeader().setVisible(False)

        # Setting font for table headers
        font = QFont()
        font.setBold(True)
        table.horizontalHeader().setFont(font)

        subjects = self.get_subject_names(rec[2])

        for i, subject in enumerate(subjects):
            marks = rec[i+3]
            grade = self.calculate_grade(marks)

            item_subject = QTableWidgetItem(subject)
            item_subject.setTextAlignment(Qt.AlignmentFlag.AlignCenter)
            table.setItem(i, 0, item_subject)

            item_marks = QTableWidgetItem(f"{marks:.2f}")
            item_marks.setTextAlignment(Qt.AlignmentFlag.AlignCenter)
            table.setItem(i, 1, item_marks)

            item_grade = QTableWidgetItem(grade)
            item_grade.setTextAlignment(Qt.AlignmentFlag.AlignCenter)
            table.setItem(i, 2, item_grade)

        total_marks_label = QLabel(f"<b>TOTAL MARKS:</b> {rec[-4]:.2f}")
        percentage_label = QLabel(f"<b>Percentage:</b> {rec[-3]:.2f}")
        grade_label = QLabel(f"<b>Grade:</b> {rec[-1]}")

        layout.addWidget(university_label)
        layout.addWidget(report_card_label)
        layout.addWidget(enrollment_label)
        layout.addWidget(name_label)
        layout.addWidget(course_label)
        layout.addWidget(separator_frame)
        layout.addWidget(table)
        layout.addWidget(total_marks_label)
        layout.addWidget(percentage_label)
        layout.addWidget(grade_label)

    def get_subject_names(self, course):
        try:
            query = f"SELECT sub1, sub2, sub3, sub4, sub5, sub6 FROM course_data WHERE course='{course}'"
            mycursor.execute(query)
            result = mycursor.fetchone()
            if result:
                subjects = [subject for subject in result]
                return subjects
            else:
                print("Course not found.")
                return []
        except Exception as e:
            print('Error:', e)
            return []

    def calculate_grade(self, marks):
        if marks >= 90:
            return "A+"
        elif 80 <= marks < 90:
            return "A"
        elif 70 <= marks < 80:
            return "B+"
        elif 60 <= marks < 70:
            return "B"
        elif 50 <= marks < 60:
            return "C+"
        elif 40 <= marks < 50:
            return "C"
        elif 30 <= marks < 40:
            return "D+"
        else:
            return "F"

class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()

        self.setWindowTitle("Student Management")
        self.setGeometry(100, 100, 450, 150)

        self.central_widget = QWidget()
        self.setCentralWidget(self.central_widget)

        self.layout = QVBoxLayout()
        self.central_widget.setLayout(self.layout)

        self.tabs = QTabWidget()

        self.student_tab = QWidget()
        self.course_tab = QWidget()
        self.exam_tab = QWidget()
        self.report_card_tab = QWidget()

        self.tabs.addTab(self.student_tab, "Student Records")
        self.tabs.addTab(self.course_tab, "Course Records")
        self.tabs.addTab(self.exam_tab, "Exam Records")
        self.tabs.addTab(self.report_card_tab, "Report Card")

        self.layout.addWidget(self.tabs)

        self.setup_student_tab()
        self.setup_course_tab()
        self.setup_exam_tab()
        self.setup_report_card_tab()

    def setup_student_tab(self):
        layout = QVBoxLayout()

        btn_add_student_record = QPushButton("Add Student Record")
        btn_display_student_record = QPushButton("Display Student Record")
        btn_modify_student_info = QPushButton("Modify Student Info")
        btn_delete_student_record = QPushButton("Delete Student Record")

        btn_add_student_record.clicked.connect(self.open_add_student_record_window)
        btn_display_student_record.clicked.connect(self.open_display_student_record_window)

        layout.addWidget(btn_add_student_record)
        layout.addWidget(btn_display_student_record)
        layout.addWidget(btn_modify_student_info)
        layout.addWidget(btn_delete_student_record)

        self.student_tab.setLayout(layout)

    def open_add_student_record_window(self):
        add_student_window = AddStudentRecordWindow()
        add_student_window.exec()

    def open_display_student_record_window(self):
        query = 'SELECT * FROM student_info'
        mycursor.execute(query)
        myrecords = mycursor.fetchall()
        display_student_window = DisplayStudentRecordWindow(myrecords)
        display_student_window.exec()

    def setup_course_tab(self):
        layout = QVBoxLayout()

        btn_add_course_record = QPushButton("Add Course Record")
        btn_display_course_record = QPushButton("Display Course Record")
        btn_modify_course_info = QPushButton("Modify Course Info")
        btn_delete_course_record = QPushButton("Delete Course Record")

        layout.addWidget(btn_add_course_record)
        layout.addWidget(btn_display_course_record)
        layout.addWidget(btn_modify_course_info)
        layout.addWidget(btn_delete_course_record)

        self.course_tab.setLayout(layout)

    def setup_exam_tab(self):
        layout = QVBoxLayout()

        btn_add_exam_record = QPushButton("Add Exam Record")
        btn_display_exam_record = QPushButton("Display Exam Record")
        btn_modify_exam_info = QPushButton("Modify Exam Info")
        btn_delete_exam_record = QPushButton("Delete Exam Record")

        layout.addWidget(btn_add_exam_record)
        layout.addWidget(btn_display_exam_record)
        layout.addWidget(btn_modify_exam_info)
        layout.addWidget(btn_delete_exam_record)

        self.exam_tab.setLayout(layout)

    def setup_report_card_tab(self):
        layout = QVBoxLayout()

        form_layout = QFormLayout()

        # Add admission number input field
        self.admno_input = QLineEdit()
        form_layout.addRow("Admission No:", self.admno_input)

        # Add submit button
        submit_button = QPushButton("Submit")
        submit_button.clicked.connect(self.display_report_card)
        form_layout.addRow(submit_button)

        self.report_card_tab.setLayout(form_layout)

    def display_report_card(self):    
        adm = self.admno_input.text()
        query = 'select * from student_record where adm_no='+"'"+adm+"'"
        mycursor.execute(query)
        rec = mycursor.fetchone()
        report_card_window = ReportCardWindow(rec)
        report_card_window.exec()            


def main():
    app = QApplication(sys.argv)
    window = MainWindow()
    window.show()
    sys.exit(app.exec())

if __name__ == "__main__":
    main()
 558 changes: 558 additions & 0 deletions558  
Report Card.py
Original file line number	Diff line number	Diff line change
@@ -0,0 +1,558 @@
import mysql.connector
mydb=mysql.connector.connect(host='localhost',user='root',passwd='root',database='reportcard')
mycursor=mydb.cursor()

def menu():
    print('\n\n\n')
    print('M E N U')
    print('1. STUDENT RECORDS')
    print('2. COURSE RECORDS')
    print('3. EXAM RECORDS')
    print('4. REPORT CARD FOR STUDENTS')
    print('5. QUIT')
    choice=int(input('Enter your choice:\t'))
    return choice

def add_student_record():
    try:
        admno=int(input('Enter admission no of student: '))
        name=input('Enter name of the student: ')
        fname=input('Enter father\'s name of the student: ')
        mname=input('Enter mother\'s name of the student: ')
        mno=int(input('Enter mobile number: '))
        address=input('Enter the address of the student: ')
        cat=input('Enter the category (general/SC/ST/OBC/other): ')
        dat=(admno,name,fname,mname,mno,address,cat)
        mycursor.execute('insert into student_info values(%s,%s,%s,%s,%s,%s,%s)',dat)
        mydb.commit()
        print('Record Added')
    except:
        print('Problem in adding student data')

def display_info_table():
    try:
        query='select * from student_info'
        mycursor.execute(query)
        myrecords=mycursor.fetchall()
        print('IGDTUW'.center(90))
        print('Student Data'.center(90))
        print()
        print(105*'-')
        print('%-5s %-15s %-15s %-15s %-15s %-15s %-15s '\
              %('admno','name','father_name','mother_name','mobile_no','address','category'))
        print(105*'-')
        for rec in myrecords:
            print('%-5s %-15s %-15s %-15s %-15s %-15s %-15s '%rec)
        print(105*'-')
    except:
        print('Something went wrong')

def display_part_student_data():
    try:
        adm=input('Enter the admission no of the student whose record is to be displayed: ')
        query='select * from student_info where admno='+"'"+str(adm)+"'"
        mycursor.execute(query)
        myrecord=mycursor.fetchone()
        print(myrecord)
        c=mycursor.rowcount
        if c==-1:
            print('nothing to display')
    except:
        print('problem in displaying particular record')

def delete_particular_student_record():
    try:        
        adm=input('Enter admission no. of the student whose record is to be deleted: ')
        query='delete from student_info where admno='+"'"+adm+"'"
        mycursor.execute(query)
        mydb.commit()
        c=mycursor.rowcount
        if c>0:
            print('Deletion done')
        else:
            print('admission no ',adm,' not found')
    except:
        print('Something went wrong')

def modify_student_info():
    try:
        adm=input('Enter admission no. of the student whose record is to be modified...')
        query='select * from student_info where admno='+"'"+adm+"'"
        mycursor.execute(query)
        myrecord=mycursor.fetchone()
        c=mycursor.rowcount
        if c==-1:
            print('Admno '+adm+' does not exist')
        else:
            name=myrecord[1]
            fname=myrecord[2]
            mname=myrecord[3]
            mob_no=myrecord[4]
            address=myrecord[5]
            category=myrecord[6]
            print('admno :',myrecord[0])
            print('name :',myrecord[1])
            print('father_name :',myrecord[2])
            print('mother_name :',myrecord[3])
            print('mobile_no :',myrecord[4])
            print('aaddress :',myrecord[5])
            print('category :',myrecord[6])
            print('----------------------')
            ch='Y'
            while True:
                print('which record to be modified?')
                print('1.Name')
                print('2.Father name')
                print('3.Mother name')
                print('4.Mobile no')
                print('5.Address')
                print('6.Category')
                c=int(input('enter the data to be modified: '))
                if c==1:
                    nname=input('enter the new name: ')
                    query='update student_info set name='+"'"+nname+"'"+'where admno='+"'"+adm+"'"
                if c==2:
                    fname=input('enter the father name: ')
                    query='update student_info set father_name='+"'"+fname+"'"+'where admno='+"'"+adm+"'"
                if c==3:
                    mname=input('enter the mother name: ')
                    query='update student_info set mother_name='+"'"+mname+"'"+'where admno='+"'"+adm+"'"
                if c==4:
                    mob=input('enter the new mobile no: ')
                    query='update student_info set mobile_no='+"'"+mob+"'"+'where admno='+"'"+adm+"'"
                if c==5:
                    adr=input('enter the new address: ')
                    query='update student_info set address='+"'"+adr+"'"+'where admno='+"'"+adm+"'"
                if c==6:
                    cat=input('enter the new category: ')
                    query='update student_info set category='+"'"+cat+"'"+'where admno='+"'"+adm+"'"
                mycursor.execute(query)
                mydb.commit()
                ch=input('want to modify anything else(y/n): ')
                if ch.upper()=='Y':
                    continue
                else:
                    break
                print('Record modified')
    except:
        print('Something went wrong')

def add_course_details():
    try:
        course=input('enter the name of the course: ')
        code=input('enter the code of the course: ')
        sub1=input('enter subject 1: ')
        sub2=input('enter subject 2: ')
        sub3=input('enter subject 3: ')
        sub4=input('enter subject 4: ')
        sub5=input('enter subject 5: ')
        sub6=input('enter subject 6: ')
        core=(course,sub1,sub2,sub3,sub4,sub5,sub6,code)
        mycursor.execute('insert into course_data values (%s,%s,%s,%s,%s,%s,%s,%s)',core)
        mydb.commit()
        print('record added')
    except:
        print('problem in adding course data')

def display_course_table():
    try:
        query='select * from course_data'
        mycursor.execute(query)
        myrecords=mycursor.fetchall()
        print('IGDTUW'.center(90))
        print('Course Record'.center(90))
        print()
        print(125*'-')
        print('%-15s %-15s %-15s %-15s %-15s %-15s %-15s %-15s'\
              %('course','sub1','sub2','sub3','sub4','sub5','sub6','c_code'))
        print(125*'-')
        for rec in myrecords:
            print('%-15s %-15s %-15s %-15s %-15s %-15s %-15s %-15s'%rec)
        print(125*'-')
    except:
        print('Something went wrong')

def display_part_course_data():
    try:
        cod=input('enter the name of the course whose record is to be displayed: ')
        query='select * from course_data where c_code='+"'"+cod+"'"
        mycursor.execute(query)
        myrecord=mycursor.fetchone()
        print(myrecord)
        c=mycursor.rowcount
        if c==-1:
            print('nothing to display')
    except:
        print('problem in displaying particular course record')

def delete_particular_course_record():
    try:        
        cod=input('Enter admission no. of the student whose record is to be deleted...')
        query='delete from course_data where c_code='+"'"+cod+"'"
        mycursor.execute(query)
        mydb.commit()
        c=mycursor.rowcount
        if c>0:
            print('Deletion done')
        else:
            print('admission no ',cod,' not found')
    except:
        print('Something went wrong')

def modify_course_data():
    try:
        cod=input('Enter code of course whose record is to be modified...')
        query='select * from course_data where c_code='+"'"+cod+"'"
        mycursor.execute(query)
        myrecord=mycursor.fetchone()
        c=mycursor.rowcount
        if c==-1:
            print('code '+cod+' does not exist')
        else:
            course=myrecord[0]
            sub1=myrecord[1]
            sub2=myrecord[2]
            sub3=myrecord[3]
            sub4=myrecord[4]
            sub5=myrecord[5]
            sub6=myrecord[6]
            code=myrecord[7]
            print('course :',myrecord[0])
            print('sub1 :',myrecord[1])
            print('sub2 :',myrecord[2])
            print('sub3 :',myrecord[3])
            print('sub4 :',myrecord[4])
            print('sub5 :',myrecord[5])
            print('sub6 :',myrecord[6])
            print('c_code :',myrecord[7])
            print('----------------------')
            ch='Y'
            while True:
                print('which record to be modified?')
                print('1.Course name')
                print('2.sub1')
                print('3.sub2')
                print('4.sub3')
                print('5.sub4')
                print('6.sub5')
                print('7.sub6')
                c=int(input('enter the data to be modified: '))
                if c==1:
                    cour=input('enter the new name: ')
                    query='update course_data set course='+"'"+cour+"'"+'where c_code='+"'"+cod+"'"
                if c==2:
                    s1=input('enter the 1st subject name: ')
                    query='update course_data set sub1='+"'"+s1+"'"+'where c_code='+"'"+cod+"'"
                if c==3:
                    s2=input('enter the 2nd subject name: ')
                    query='update course_data set sub2='+"'"+s2+"'"+'where c_code='+"'"+cod+"'"
                if c==4:
                    s3=input('enter the 3rd subject name: ')
                    query='update course_data set sub3='+"'"+s3+"'"+'where c_code='+"'"+cod+"'"
                if c==5:
                    s4=input('enter the 4th subject name: ')
                    query='update course_data set sub4='+"'"+s4+"'"+'where c_code='+"'"+cod+"'"
                if c==6:
                    s5=input('enter the 5th subject name: ')
                    query='update course_data set sub5='+"'"+s5+"'"+'where c_code='+"'"+cod+"'"
                if c==7:
                    s6=input('enter the 6th subject name: ')
                    query='update course_data set sub6='+"'"+s6+"'"+'where c_code='+"'"+cod+"'"
                mycursor.execute(query)
                mydb.commit()
                ch=input('want to modify anything else(y/n): ')
                if ch.upper()=='Y':
                    continue
                else:
                    break
                print('Record modified')
    except:
        print('Something went wrong')

def add_student_score():
    try:
        print('Enter students information')
        admno=int(input('enter admission no of student'))
        name=input('enter name of the student')
        course=input('enter course: ')
        phy=int(input('Enter marks in SUB1: '))
        chem=int(input('Enter marks in SUB2: '))
        maths=int(input('Enter marks in SUB3: '))
        cs=int(input('Enter marks in SUB4: '))
        coa=int(input('Enter marks in SUB5: '))
        pyt=int(input('Enter marks in SUB6: '))
        wd=int(input('Enter the number of working days...'))
        ad=int(input('Enter the number of days student attended the class...'))
        r=(ad/wd)*100
        atten=round(r,2)
        total=phy+chem+maths+cs+coa+pyt
        perc=total/6
        if perc>=90:
            grade='A'
        elif perc>=80:
            grade='B'
        elif perc>=70:
            grade='C'
        elif perc>=60:
            grade='D'
        elif perc>=50:
            grade='E'
        else:
            grade='F'
        rec=(admno,name,course,phy,chem,maths,cs,coa,pyt,total,perc,atten,grade)
        mycursor.execute('insert into student_record values (%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)',rec)
        mydb.commit()
        print('Record added')
    except:
        print('problem in adding student scores')

def display_part_student_record():
    try:
        adm=input('enter the admission no of the student whose record is to be displayed: ')
        query='select * from student_record where adm_no='+"'"+adm+"'"
        mycursor.execute(query)
        myrecord=mycursor.fetchone()
        print(myrecord)
        c=mycursor.rowcount
        if c==-1:
            print('nothing to display')
    except:
        print('problem in displaying particular student score record')

def display_score_table():
    try:
        query='select * from student_record'
        mycursor.execute(query)
        myrecords=mycursor.fetchall()
        print('IGDTUW'.center(90))
        print('Student Record'.center(90))
        print()
        print(135*'-')
        print('%-5s %-15s %-8s %-8s  %-8s  %-8s  %-8s  %-8s  %-8s  %-9s %-8s %12s %12s'\
              %('admno','name','course','sub1','sub2','sub3','sub4','sub5','sub6','total','perc','attendance','div'))
        print(135*'-')
        for rec in myrecords:
            print('%-5s %-15s %-8s %-8s  %-8s  %-8s  %-8s  %-8s  %-8s  %-9s %-8s %12s %12s'%rec)
        print(135*'-')
    except:
        print('Something went wrong')

def delete_particular_score_record():
    try:        
        adm=input('Enter admission no. of the student whose record is to be deleted...')
        query='delete from student_record where adm_no='+adm
        mycursor.execute(query)
        mydb.commit()
        c=mycursor.rowcount
        if c>0:
            print('Deletion done')
        else:
            print('admission no ',adm,' not found')
    except:
        print('Something went wrong')

def modify_score_record():
    try:
        adm=input('Enter admission no. of the student whose record is to be modified...')
        query='select * from student_record where adm_no='+"'"+adm+"'"
        mycursor.execute(query)
        myrecord=mycursor.fetchone()
        c=mycursor.rowcount
        if c==-1:
            print('Admno '+adm+' does not exist')
        else:
            name=myrecord[1]
            course=myrecord[2]
            sub1=myrecord[3]
            sub2=myrecord[4]
            sub3=myrecord[5]
            sub4=myrecord[6]
            sub5=myrecord[7]
            sub6=myrecord[8]
            att=myrecord[11]
            print('admno :',myrecord[0])
            print('name :',myrecord[1])
            print('course :',myrecord[2])
            print('sub1 :',myrecord[3])
            print('sub2 :',myrecord[4])
            print('sub3 :',myrecord[5])
            print('sub4 :',myrecord[6])
            print('sub5 :',myrecord[7])
            print('sub6 :',myrecord[8])
            print('total :',myrecord[9])
            print('percentage :',myrecord[10])
            print('attendance :',myrecord[11])
            print('grade :',myrecord[12])
            print('----------------------')
            ch='Y'
            while True:
                print('which record to be modified?')
                print('1.Name')
                print('2.Course')
                print('3.Marks')
                print('4.Attendance')
                c=int(input('enter the data to be modified: '))
                if c==1:
                    nname=input('enter the new name: ')
                    query='update student_record set name='+"'"+nname+"'"+'where adm_no='+"'"+adm+"'"
                if c==2:
                    ncour=input('enter the new course:')
                    query='update student_record set course='+"'"+ncour+"'"+'where adm_no='+"'"+adm+"'"
                if c==3:
                    print('1.sub1')
                    print('2.sub2')
                    print('3.sub3')
                    print('4.sub4')
                    print('5.sub5')
                    print('6.sub6')
                    c1=int(input('enter the subject whose marks to be updated: '))
                    if c1==1:
                        sub1=int(input('enter new marks in sub1: '))
                        query='update student_record set sub1='+"'"+str(sub1)+"'"+'where adm_no='+"'"+adm+"'"
                    if c1==2:
                        sub2=int(input('enter new marks in sub2: '))
                        query='update student_record set sub2='+"'"+str(sub2)+"'"+'where adm_no='+"'"+adm+"'"
                    if c1==3:
                        sub3=int(input('enter new marks in sub3: '))
                        query='update student_record set sub3='+"'"+str(sub3)+"'"+'where adm_no='+"'"+adm+"'"
                    if c1==4:
                        sub4=int(input('enter new marks in sub4: '))
                        query='update student_record set sub4='+"'"+str(sub4)+"'"+'where adm_no='+"'"+adm+"'"
                    if c1==5:
                        sub5=int(input('enter new marks in sub5: '))
                        query='update student_record set sub5='+"'"+str(sub5)+"'"+'where adm_no='+"'"+adm+"'"
                    if c1==6:
                        sub6=int(input('enter new marks in sub6: '))
                        query='update student_record set sub6='+"'"+str(sub6)+"'"+'where adm_no='+"'"+adm+"'"
                if c==4:
                    a=int(input('enter total number of working days: '))
                    b=int(input('enter number of days student attended the classes: '))
                    att=(b/a)*100
                    query='update student_record set attendance='+"'"+str(att)+"'"+'where adm_no='+"'"+adm+"'"
                total=sub1+sub2+sub3+sub4+sub5+sub6
                perc=total/6
                if perc>=90:
                    grade='A'
                elif perc>=80:
                    grade='B'
                elif perc>=70:
                    grade='C'
                elif perc>=60:
                    grade='D'
                elif perc>=50:
                    grade='E'
                else:
                    grade='F'
                query1='update student_record set total='+"'"+str(total)+"'"+','+'percentage='+"'"+str(perc)+"'"+','+\
                       'grade='+"'"+str(grade)+"'"+'where adm_no='+adm
                mycursor.execute(query1)
                rec =(adm,name,course,sub1,sub2,sub3,sub4,sub5,sub6,total,perc,att,grade)
                mycursor.execute(query)
                mydb.commit()
                ch=input('Want to modify anything else(y/n): ')
                if ch.upper()=='Y':
                    continue
                else:
                    break
                print('Record modified')
    except:
        print('Something went wrong')

def menu2():
    print('1 for CREATE')
    print('2 for DISPLAY A PARTICULAR RECORD')
    print('3 for UPDATE')
    print('4 for DELETION')
    ch1=int(input('enter the choice in student records (1/2/3/4):\n'))
    return ch1

def get_subject_names(course):
    try:
        query = f"SELECT sub1, sub2, sub3, sub4, sub5, sub6 FROM course_data WHERE course='{course}'"
        mycursor.execute(query)
        result = mycursor.fetchone()
        if result:
            subjects = [subject for subject in result]
            return subjects
        else:
            print("Course not found.")
            return []
    except Exception as e:
        print('Error:', e)
        return []

def report_card_particular():
    try:
        adm = input('Enter admission no. of student to get the record card: ')
        query = f"SELECT * FROM student_record WHERE adm_no={adm}"
        mycursor.execute(query)
        rec = mycursor.fetchone()

        if rec:
            print('################################################################################################')
            print()
            print('----------------'.center(90))
            print('SRM University'.center(90))
            print('Report Card'.center(90))
            print('----------------'.center(90))
            print()
            print('Enrollment no.: %4d                   Name:%-15s            Course:%-15s '%(rec[0], rec[1], rec[2]))
            print()

            subjects = get_subject_names(rec[2])

            print('Subject'.ljust(50), 'Marks')
            print(135*'-')

            for i, subject in enumerate(subjects):
                print(subject.ljust(50), ":", f"{rec[i+3]:>9.2f}")

            print(135*'-')

            print()
            print('TOTAL MARKS:  %9.2f                  Percentage:  %9.2f        Grade:  %-5s'%(rec[-4], rec[-3], rec[-1]))
            print()
            print('################################################################################################\n')               
        else:
            print('Nothing to display')
    except Exception as e:
        print('Something went wrong:', e)

while True:
    choice=menu()
    if choice==1:
        ch1=menu2()
        if ch1==1:
            add_student_record()
        if ch1==2:
            display_part_student_data()
        if ch1==3:
            modify_student_info()
        if ch1==4:
            delete_particular_student_record()
    if choice==2:
        ch1=menu2()
        if ch1==1:
            add_course_details()
        if ch1==2:
            display_part_course_data()
        if ch1==3:
            modify_course_data()
        if ch1==4:
            delete_particular_course_record()
    if choice==3:
        ch1=menu2()
        if ch1==1:
            add_student_score()
        if ch1==2:
            display_part_student_record()
        if ch1==3:
            modify_score_record()
        if ch1==4:
            delete_particular_score_record()
    if choice==4:
        report_card_particular()
    if choice==5:
        print('EXITING PROGRAM')
        print('GOODBYE')
        break
 39 changes: 39 additions & 0 deletions39  
Report Card.sql
Original file line number	Diff line number	Diff line change
@@ -0,0 +1,39 @@
-- Create table for student information
CREATE TABLE student_info (
    admno INT PRIMARY KEY,
    name VARCHAR(50),
    father_name VARCHAR(50),
    mother_name VARCHAR(50),
    mobile_no BIGINT,
    address VARCHAR(100),
    category VARCHAR(20)
);

-- Create table for course data
CREATE TABLE course_data (
    course VARCHAR(50),
    sub1 VARCHAR(50),
    sub2 VARCHAR(50),
    sub3 VARCHAR(50),
    sub4 VARCHAR(50),
    sub5 VARCHAR(50),
    sub6 VARCHAR(50),
    c_code VARCHAR(10) PRIMARY KEY
);

-- Create table for student scores
CREATE TABLE student_record (
    adm_no INT PRIMARY KEY,
    name VARCHAR(255),
    course VARCHAR(255),
    sub1 INT,
    sub2 INT,
    sub3 INT,
    sub4 INT,
    sub5 INT,
    sub6 INT,
    total FLOAT,
    percentage FLOAT,
    attendance FLOAT,
    grade VARCHAR(10)
);
0 comments on commit 0d12f8c
Please sign in to comment.
Footer
© 2024 GitHub, Inc.
Footer navigation
Terms
Privacy
Security
Status
Docs
Contact
Manage cookies
Do not share my personal information
