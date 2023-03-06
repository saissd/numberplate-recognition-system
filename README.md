# numberplate-recognition-system
from tkinter import *
import tkinter as tk
from tkinter import filedialog
from tkinter.filedialog import askopenfile
from PIL import Image, ImageTk
import cv2
from matplotlib import pyplot as plt
import numpy as np
import imutils
import easyocr
from tkinter import messagebox

r3 = tk.Tk()
r3.geometry('925x500+300+268')
r3.configure(bg="#fff")
r3.title('LOGIN')
r3_font1 = ('times', 18, 'bold')

lr1 = tk.Label(r3, text='LOGIN', width=30, font=r3_font1, bg="red", fg="white")
lr1.grid(row=1, ipadx=5)
C1 = tk.Button(r3, text='ADMIN', height=3, width=20, command=lambda: admin())
C1.grid(row=3, column=0)
C2 = tk.Button(r3, text="USER", height=3, width=20, command=lambda: user())
C2.grid(row=3, column=1)


def admin():
    r1 = Toplevel(r3)
    r1.title('ADMIN_Login')
    r1.geometry('925x500+300+268')
    r1.configure(bg="#fff")
    r1.resizable(False, False)

    def signin():
        username = user.get()
        password = code.get()
        if username == 'admin' and password == '1234':
            s = Toplevel(r1)
            s.title("app")
            s.geometry('925x500+300+268')
            s.config(bg="white")
            Label(s, text='Hello Everyone!', bg='#fff', font=('Calibri (Body)', 50, 'bold')).pack(expand=True)
            s.mainloop()
        elif username != 'admin' and password != '1234':
            messagebox.showerror("Invalid", "invalid username and password")
        elif password != '1234':
            messagebox.showerror("Invalid", "invalid password")
        elif username != 'admin':
            messagebox.showerror("Invalid", "invalid username")

    frame = Frame(r1, width=350, height=350, bg="white")
    frame.place(x=480, y=70)

    heading = Label(frame, text='Sign in', fg='#57a1f8', bg="white", font=('Microsoft vale UE Light', 23, 'bold'))
    heading.place(x=100, y=5)

    #########_______________________
    def on_enter(e):
        user.delete(0, 'end')

    def on_leave(e):
        name = user.get()
        if name == '':
            user.insert(0, "Username")

    user = Entry(frame, width=25, fg="black", border=0, bg="white", font=('Microsoft YaHes UI Light', 11))
    user.place(x=30, y=80)
    user.insert(0, 'Username:')
    user.bind('<FocusIn>', on_enter)
    user.bind('<FocusOut>', on_leave)

    Frame(frame, width=295, height=2, bg="black").place(x=25, y=107)

    #########----------------
    def on_enter(e):
        code.delete(0, 'end')

    def on_leave(e):
        name = code.get()
        if name == '':
            code.insert(0, "password")

    code = Entry(frame, width=25, fg="black", border=0, bg="white", font=('Microsoft VaHei UI Light', 11))
    code.place(x=30, y=150)
    code.insert(0, 'Password')
    code.bind('<FocusIn>', on_enter)
    code.bind('<FocusOut>', on_leave)

    Frame(frame, width=295, height=2, bg="black").place(x=25, y=177)
    ########---------------------
    Button(frame, width=39, pady=7, text='sign in', bg='#57a1f8', fg='white', border=0, command=signin).place(x=35,
                                                                                                              y=204)
    label = Label(frame, text="Don't have an account?", fg="black", bg='white', font=('Microsoft YaHei UI Light', 9))
    label.place(x=75, y=270)

    sign_up = tk.Button(frame, width=6, text='Sig up', border=0, bg='white', cursor='hand2', fg='#57a1f8')
    sign_up.place(x=215, y=270)

    r1.mainloop()


def user():
    r2 = Toplevel(r3)
    r2.title('USER_LOGIN')
    r2.geometry('925x500+300+268')
    r2.configure(bg="#fff")
    r2.resizable(False, False)

    def sign_in():
        username = user.get()
        password = code.get()
        if username == 'user' and password == '1234':
            my_w = Toplevel(r2)
            my_w.geometry('925x500+300+268')  # Size of the window
            my_w.title('numberplate detection system')
            my_font1 = ('times', 18, 'bold')
            l1 = tk.Label(my_w, text='NUMBERPLATE DETECTION', width=30, font=my_font1, bg="red", fg="white")
            l1.grid(row=1, ipadx=5)
            b1 = tk.Button(my_w, text='TAKE IMAGE', height=3, width=20, command=lambda: upload_file())
            b1.grid(row=7, column=0)

            def upload_file():

                global img
                f_types = [('Jpg Files', '*.jpg')]
                filename = filedialog.askopenfilename(filetypes=f_types)
                img = ImageTk.PhotoImage(file=filename)
                a = filename.split('/')
                b = len(a)

                b2 = tk.Button(my_w, image=img)  # using Button
                b2.grid(row=3, column=0)
                b3 = tk.Button(my_w, text="SCAN", height=2, width=20, command=lambda: output(a[b - 1]))
                b3.grid(row=7, column=1)

                def clear():
                    b2.delete(END)

                def output(c):

                    img = cv2.imread(c)
                    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

                    bfilter = cv2.bilateralFilter(gray, 11, 17, 17)  # Noise reduction
                    edged = cv2.Canny(bfilter, 30, 200)  # Edge detection
                    # plt.imshow(cv2.cvtColor(edged, cv2.COLOR_BGR2RGB))

                    keypoints = cv2.findContours(edged.copy(), cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
                    contours = imutils.grab_contours(keypoints)
                    contours = sorted(contours, key=cv2.contourArea, reverse=True)[:10]

                    location = None
                    for contour in contours:
                        approx = cv2.approxPolyDP(contour, 10, True)
                        if len(approx) == 4:
                            location = approx
                            break

                    mask = np.zeros(gray.shape, np.uint8)
                    new_image = cv2.drawContours(mask, [location], 0, 255, -1)
                    new_image = cv2.bitwise_and(img, img, mask=mask)

                    # plt.imshow(cv2.cvtColor(new_image, cv2.COLOR_BGR2RGB))

                    (x, y) = np.where(mask == 255)
                    (x1, y1) = (np.min(x), np.min(y))
                    (x2, y2) = (np.max(x), np.max(y))
                    cropped_image = gray[x1:x2 + 1, y1:y2 + 1]

                    # plt.imshow(cv2.cvtColor(cropped_image, cv2.COLOR_BGR2RGB))

                    reader = easyocr.Reader(['en'])
                    result = reader.readtext(cropped_image)
                    r1 = result[0][1]
                    print(result)
                    label1 = Label(my_w, text="REGISTRATION NUMBER", height=2, width=25, fg="red").place(x=559, y=50)
                    label2 = Label(my_w, text="ACCURACY SCORE", height=2, width=25, fg="red").place(x=550, y=200)

                    T = tk.Text(my_w, height=3, width=25)
                    T.grid(row=3, column=3)
                    T.insert(tk.END, result[0][1])

                    T1 = tk.Text(my_w, height=3, width=25)
                    T1.grid(row=5, column=3)
                    T1.insert(tk.END, result[0][2])

            my_w.mainloop()
        elif username != 'user' and password != '1234':
            messagebox.showerror("Invalid", "invalid username and password")
        elif password != '1234':
            messagebox.showerror("Invalid", "invalid password")
        elif username != 'user':
            messagebox.showerror("Invalid", "invalid username")

    frame = Frame(r2, width=350, height=350, bg="white")
    frame.place(x=480, y=70)

    heading = Label(frame, text='Sign in', fg='#57a1f8', bg="white", font=('Microsoft vale UE Light', 23, 'bold'))
    heading.place(x=100, y=5)

    #########_______________________
    def on_enter(e):
        user.delete(0, 'end')

    def on_leave(e):
        name = user.get()
        if name == '':
            user.insert(0, "Username")

    user = Entry(frame, width=25, fg="black", border=0, bg="white", font=('Microsoft YaHes UI Light', 11))
    user.place(x=30, y=80)
    user.insert(0, 'Username:')
    user.bind('<FocusIn>', on_enter)
    user.bind('<FocusOut>', on_leave)

    Frame(frame, width=295, height=2, bg="black").place(x=25, y=107)

    #########----------------
    def on_enter(e):
        code.delete(0, 'end')

    def on_leave(e):
        name = code.get()
        if name == '':
            code.insert(0, "password")

    code = Entry(frame, width=25, fg="black", border=0, bg="white", font=('Microsoft VaHei UI Light', 11))
    code.place(x=30, y=150)
    code.insert(0, 'Password')
    code.bind('<FocusIn>', on_enter)
    code.bind('<FocusOut>', on_leave)

    Frame(frame, width=295, height=2, bg="black").place(x=25, y=177)
    ########---------------------
    Button(frame, width=39, pady=7, text='sign in', bg='#57a1f8', fg='white', border=0, command=sign_in).place(x=35,
                                                                                                               y=204)
    label = Label(frame, text="Don't have an account?", fg="black", bg='white', font=('Microsoft YaHei UI Light', 9))
    label.place(x=75, y=270)

    sign_up = tk.Button(frame, width=6, text='Sig up', border=0, bg='white', cursor='hand2', fg='#57a1f8')
    sign_up.place(x=215, y=270)

    r2.mainloop()


r3.mainloop()
