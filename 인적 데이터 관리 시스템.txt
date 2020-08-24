import pymysql
import tkinter as tk
import tkinter.messagebox
import re

# 컴퓨터에 있는 MySQL에 연결한다.
db = pymysql.connect(host= 'localhost', user='root', passwd='0000', db='peopledata', charset='utf8')
curs = db.cursor()

# Tkinter화면을 만들고 크기와 배경색을 설정한다.
window=tk.Tk()
window.title("인적 데이터 관리 시스템")
window.geometry("200x200")
window.resizable(False, False)
window.configure(background='white')

# 손님이 저장되어있을 경우 방문 횟수를 1회 증가시킨다.
def increase():
    name = str1.get()
    birth = str2.get()
    sql = 'update people set visit = visit+1 where name = \'{}\' and birth = \'{}\''.format(name,birth)
    curs.execute(sql)
    db.commit()

# 손님이 방문 한 적이 없는 경우 데이터베이스에 손님의 이름과 생일 방문 횟수 1을 저장한다.
def create():
    name = str1.get()
    birth = str2.get()
    sql = 'insert into people(name,birth,visit) values(\'{}\',\'{}\',\'{}\')'.format(name,birth,1)
    curs.execute(sql)
    db.commit()

# 데이터베이스에 저장하는지 묻는 메세지박스를 출력해준다.
def how():
    msgbox = tk.messagebox.askquestion('','데이터베이스에 저장하시겠습니까?',icon = 'error')
    if msgbox == 'yes':
        create()
        tk.messagebox.showinfo('','생성')
    else:
        tk.messagebox.showinfo('','생성하지않음')

# 방문 횟수를 증가시키는 메세지 박스를 출력해준다.
def how_increase():
    msgbox = tk.messagebox.askquestion('','방문 횟수를 증가시키겠습니까??',icon = 'error')
    if msgbox == 'yes':
        increase()
        tk.messagebox.showinfo('','증가')
    else:
        tk.messagebox.showinfo('','그대로')

# 찾기 버튼을 눌렀을 때 실행되는 함수
def select():
    window.configure(background='white')
    name = str1.get()
    birth = str2.get()
    sql = 'select visit from people where name = \'{}\' and birth = \'{}\''.format(name,birth)
    curs.execute(sql)
    rows = str(curs.fetchone())

    try:
        visits = int(re.findall('([0-9]+)',rows)[0]) # 정규식을 이용해 숫자를 찾아낸다
    except: # 숫자가 없을 경우
        label1['text'] = '없음'
        label2['text'] = '없음'
        label3['text'] = '처음 방문 함'
        how()
        return
        
    label1['text'] = name
    label2['text'] = birth
    label3['text'] = str(visits)
    
    if int(visits)%5 == 0: # 방문 횟수 5회일 경우 이벤트 대상자임을 알린다.
        window.configure(background='red') # 화면이 빨간색으로 변한다.
        tk.messagebox.showinfo('','이벤트 대상자')
    
# str1과 str2를 정의하는 부분   
str1 = tk.StringVar() # 이름
textbox1 = tk.Entry(window, width = 20, textvariable=str1)
textbox1.grid()

str2 = tk.StringVar() # 생일
textbox2 = tk.Entry(window, width = 20, textvariable=str2)
textbox2.grid()

# 찾기 버튼을 정의하는 부분
button = tk.Button(window,width = 20, command=select, text = '찾기')
button.grid()

label1 = tk.Label(window,text = '이름')
label2 = tk.Label(window,text = '생일')
label3 = tk.Label(window,text = '방문 횟수')

label1.grid()
label2.grid()
label3.grid()

button2 = tk.Button(window,width = 20, command = how_increase , text = '방문 횟수 증가').grid()

window.mainloop()
db.close()
