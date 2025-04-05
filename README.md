import tkinter as tk
from tkinter import messagebox, PhotoImage, ttk
from PIL import Image, ImageTk
import sqlite3
import time
import os


# Paths for Images
BG_PATH = r"C:\Rashi\College Assignments\SEM 4\MPR PYTHON\bg.png"
LOGO_PATH = r"C:\Rashi\College Assignments\SEM 4\MPR PYTHON\brightland_logo.png.webp"
POOL_PATH = r"C:\Rashi\College Assignments\SEM 4\MPR PYTHON\pool.png"

# Database Setup
def setup_db():
    conn = sqlite3.connect("hotel.db")
    cursor = conn.cursor()
    cursor.execute('''CREATE TABLE IF NOT EXISTS users (
                        id INTEGER PRIMARY KEY AUTOINCREMENT, 
                        username TEXT UNIQUE, 
                        password TEXT)''')
    cursor.execute('''CREATE TABLE IF NOT EXISTS rooms (
                        room_id INTEGER PRIMARY KEY,
                        room_type TEXT,
                        price REAL)''')
    cursor.execute('''CREATE TABLE IF NOT EXISTS customers (
                        customer_ref TEXT PRIMARY KEY,
                        customer_name TEXT,
                        gender TEXT,
                        postcode TEXT,
                        mobile TEXT,
                        email TEXT,
                        nationality TEXT,
                        id_proof_type TEXT,
                        id_number TEXT,
                        address TEXT)''')
    conn.commit()
    conn.close()

# Dashboard Window
def dashboard():
    global dash  # Use the global variable
    dash = tk.Tk()
    dash.title("Brightland Resorts & Spa")
    dash.geometry("1000x600")

    # Rest of your dashboard code


    # Background Pool Image
    if os.path.exists(POOL_PATH):
        pool_image = Image.open(POOL_PATH).resize((1300, 600))
        pool_photo = ImageTk.PhotoImage(pool_image)
        pool_label = tk.Label(dash, image=pool_photo)
        pool_label.image = pool_photo
        pool_label.place(x=0, y=0)

    # Header Section
    header_frame = tk.Frame(dash, bg="black")
    header_frame.place(relwidth=1, relheight=0.12)

    # Logo Image
    if os.path.exists(LOGO_PATH):
        logo_image = Image.open(LOGO_PATH).resize((100, 100))
        logo_photo = ImageTk.PhotoImage(logo_image)
        logo_label = tk.Label(header_frame, image=logo_photo, bg="black")
        logo_label.image = logo_photo
        logo_label.pack(side="left", padx=10)

    title_label = tk.Label(header_frame, text="BRIGHTLAND RESORTS & SPA", font=("Times New Roman", 30, "bold"), fg="gold", bg="black")
    title_label.pack(side="top", pady=10)

    # Sidebar Menu
    sidebar_frame = tk.Frame(dash, bg="black")
    sidebar_frame.place(relwidth=0.2, relheight=0.80, rely=0.12)

    menu_label = tk.Label(sidebar_frame, text="MENU", font=("Times New Roman", 16, "bold"), fg="gold", bg="black")
    menu_label.pack(pady=10)

    menu_buttons = ["CUSTOMER", "BOOKING", "ROOMS", "DETAILS", "LOGOUT"]

    def on_button_click(name):
        if name == "CUSTOMER":
            add_customer_window()
        elif name == "BOOKING":
            booking_window()
        elif name == "ROOMS":
            room_management_window()
        elif name == "LOGOUT":
            dash.destroy()
        else:
            print(f"{name} button clicked")

    for name in menu_buttons:
        button = tk.Button(sidebar_frame, text=name, font=("Arial", 12, "bold"), bg="#007bff", fg="white", width=15, height=2, command=lambda n=name: on_button_click(n))
        button.pack(pady=5)

    # Offers Section
    offers_frame = tk.Frame(dash, bg="#ffcc00")
    offers_frame.place(relwidth=1, relheight=0.15, rely=0.85)

    offer_label = tk.Label(offers_frame, text="\u273F Special Offers \u273F\n- Get 20% off on all Deluxe Rooms for a limited time!\n- Book an Executive Room for 2 nights and enjoy 1 night free!\n- Book a Suite and enjoy a luxury spa experience on the house!", font=("Arial", 14, "bold"), fg="black", bg="#ffcc00")
    offer_label.pack()

    dash.mainloop()

    



# Add Room
def room_management_window():
    room_win = tk.Toplevel()
    room_win.title("Add Room Management Details")
    room_win.geometry("1200x650")
    room_win.configure(bg="#f2f2f2")


def open_details(self):
        if not hasattr(self, 'details') or not self.details.winfo_exists():
            self.details = RoomDetails(self.main_frame)
def add_room():
    room_id = room_no_entry.get()
    room_type = room_type_var.get()

    room_prices = {
        'Single': 2000.00,
        'Double': 3000.00,
        'Luxury': 5000.00,
        'Suite': 8000.00,
        'Duplex': 10000.00
    }
    price = room_prices.get(room_type, 2000.00)

    if room_id and room_type:
        try:
            conn = sqlite3.connect('hotel.db')
            cursor = conn.cursor()
            cursor.execute("INSERT INTO rooms (room_id, room_type, price, status) VALUES (?, ?, ?, ?)", 
                            (int(room_id), room_type, price, 'Available'))
            conn.commit()
            conn.close()
            messagebox.showinfo('Success', 'Room added successfully!')
            display_rooms()
            reset_fields()
        except Exception as e:
            messagebox.showerror('Error', str(e))
    else:
        messagebox.showerror('Error', 'Room ID and Room Type are required!')

# Display Rooms
def display_rooms():
    for row in room_tree.get_children():
        room_tree.delete(row)

    conn = sqlite3.connect('hotel.db')
    cursor = conn.cursor()
    cursor.execute("SELECT room_id, room_type, price, status FROM rooms")

    for row in cursor.fetchall():
        status = row[3]
        if status == 'Available':
            tag = 'available'
        elif status == 'Not Available':
            tag = 'not_available'
        else:
            tag = 'unknown_status'
        room_tree.insert('', 'end', values=row, tags=(tag,))
    conn.close()

# Reset Fields
def reset_fields():
    room_no_entry.delete(0, tk.END)
    room_type_var.set('Single')

# Mark room as Not Available (Booked)
def book_room():
    selected = room_tree.selection()
    if selected:
        room_id = room_tree.item(selected[0])['values'][0]
        conn = sqlite3.connect('hotel.db')
        cursor = conn.cursor()
        cursor.execute("UPDATE rooms SET status = 'Not Available' WHERE room_id = ?", (room_id,))
        conn.commit()
        conn.close()
        display_rooms()

# Mark room as Available (Vacant)
def vacate_room():
    selected = room_tree.selection()
    if selected:
        room_id = room_tree.item(selected[0])['values'][0]
        conn = sqlite3.connect('hotel.db')
        cursor = conn.cursor()
        cursor.execute("UPDATE rooms SET status = 'Available' WHERE room_id = ?", (room_id,))
        conn.commit()
        conn.close()
        display_rooms()

# UI Setup
window = tk.Tk()
window.title('Room Booking Details')
window.geometry('950x600')
window.config(bg='#f0f0f5')

setup_db()

# Header
header_frame = tk.Frame(window, bg='#2c3e50', height=80)
header_frame.pack(side='top', fill='x')
header_label = tk.Label(header_frame, text='ROOM BOOKING DETAILS', font=('Arial', 24, 'bold'), fg='white', bg='#2c3e50')
header_label.pack(pady=20)

# Form
form_frame = tk.Frame(window, bg='white', bd=2, relief='ridge')
form_frame.place(x=20, y=100, width=400, height=300)

# Add Room UI
tk.Label(form_frame, text='New Room Add', font=('Arial', 14, 'bold'), bg='white').grid(row=0, columnspan=2, pady=10)
tk.Label(form_frame, text='Room No:', font=('Arial', 12), bg='white').grid(row=1, column=0, pady=10, padx=10)
room_no_entry = tk.Entry(form_frame, font=('Arial', 12))
room_no_entry.grid(row=1, column=1)

room_type_var = tk.StringVar(value='Single')
tk.Label(form_frame, text='Room Type:', font=('Arial', 12), bg='white').grid(row=2, column=0, pady=10, padx=10)
room_type_menu = ttk.Combobox(form_frame, textvariable=room_type_var, values=['Single', 'Double', 'Luxury', 'Suite', 'Duplex'], state='readonly', font=('Arial', 12))
room_type_menu.grid(row=2, column=1)

# Buttons
button_frame = tk.Frame(form_frame, bg='white')
button_frame.grid(row=3, columnspan=2, pady=20)
tk.Button(button_frame, text='Add', command=add_room, width=10, bg='#27ae60', fg='white', font=('Arial', 12)).grid(row=0, column=0, padx=5)
tk.Button(button_frame, text='Reset', command=reset_fields, width=10, bg='#e67e22', fg='white', font=('Arial', 12)).grid(row=0, column=1, padx=5)

# Table
table_frame = tk.Frame(window, bg='white', bd=2, relief='ridge')
table_frame.place(x=450, y=100, width=470, height=450)

room_tree = ttk.Treeview(table_frame, columns=('Room ID', 'Room Type', 'Price', 'Status'), show='headings', height=15)
room_tree.heading('Room ID', text='Room ID')
room_tree.heading('Room Type', text='Room Type')
room_tree.heading('Price', text='Price')
room_tree.heading('Status', text='Status')

room_tree.column('Room ID', width=80, anchor='center')
room_tree.column('Room Type', width=100, anchor='center')
room_tree.column('Price', width=80, anchor='center')
room_tree.column('Status', width=100, anchor='center')

room_tree.tag_configure('available', background='lightgreen')
room_tree.tag_configure('not_available', background='lightcoral')
room_tree.tag_configure('unknown_status', background='khaki')
room_tree.pack(fill='both', expand=True)

# Room Action Buttons
action_frame = tk.Frame(window, bg='#f0f0f5')
action_frame.place(x=450, y=560)
tk.Button(action_frame, text='Book Room', command=book_room, width=15, bg='#2980b9', fg='white', font=('Arial', 12)).grid(row=0, column=0, padx=5)
tk.Button(action_frame, text='Vacate Room', command=vacate_room, width=15, bg='#27ae60', fg='white', font=('Arial', 12)).grid(row=0, column=1, padx=5)

# Load initial data
display_rooms()

window.mainloop()


import tkinter as tk
from tkinter import ttk

def add_customer_window():
    customer_win = tk.Toplevel()
    customer_win.title("Add Customer Details")
    customer_win.geometry("1200x650")
    customer_win.configure(bg="#f2f2f2")

    # Header
    header_frame = tk.Frame(customer_win, bg="#2c3e50", height=80)
    header_frame.pack(side="top", fill="x")

    title_label = tk.Label(
        header_frame,
        text="➤ Add Customer Details",
        font=("Segoe UI", 24, "bold"),
        fg="#f1c40f",
        bg="#2c3e50"
    )
    title_label.pack(pady=20)

    # Left Form Frame
    form_frame = tk.LabelFrame(
        customer_win, text="Customer Information",
        font=("Segoe UI", 14, "bold"),
        bg="white", fg="#2c3e50",
        padx=20, pady=10
    )
    form_frame.place(x=30, y=100, width=520, height=500)

    labels = [
        "Customer Ref:", "Customer Name:", "Gender:", "PostCode:",
        "Mobile:", "Email:", "Nationality:", "Id Proof Type:",
        "Id Number:", "Address:"
    ]

    entries = {}

    for i, label in enumerate(labels):
        lbl = tk.Label(form_frame, text=label, font=("Segoe UI", 11), bg="white")
        lbl.grid(row=i, column=0, sticky="w", pady=6)

        if label == "Gender:":
            gender_var = tk.StringVar(value="Male")
            gender_dropdown = ttk.Combobox(form_frame, textvariable=gender_var,
                                           values=["Male", "Female", "Other"], state="readonly")
            gender_dropdown.grid(row=i, column=1, padx=10, pady=5)
            entries[label] = gender_var

        elif label == "Nationality:":
            nationality_var = tk.StringVar(value="Indian")
            nationality_dropdown = ttk.Combobox(form_frame, textvariable=nationality_var,
                                                values=["Indian", "American", "Other"], state="readonly")
            nationality_dropdown.grid(row=i, column=1, padx=10, pady=5)
            entries[label] = nationality_var

        elif label == "Id Proof Type:":
            proof_var = tk.StringVar(value="Aadhar Card")
            proof_dropdown = ttk.Combobox(form_frame, textvariable=proof_var,
                                          values=["Aadhar Card", "Passport", "Driving License"], state="readonly")
            proof_dropdown.grid(row=i, column=1, padx=10, pady=5)
            entries[label] = proof_var

        elif label == "Address:":
            entry = tk.Text(form_frame, width=22, height=4, font=("Segoe UI", 10))
            entry.grid(row=i, column=1, padx=10, pady=5)
            entries[label] = entry
        else:
            entry = tk.Entry(form_frame, font=("Segoe UI", 10))
            entry.grid(row=i, column=1, padx=10, pady=5)
            entries[label] = entry

    # Action Buttons Frame
    btn_frame = tk.Frame(customer_win, bg="#f2f2f2")
    btn_frame.place(x=30, y=610)

    button_style = {
        "font": ("Segoe UI", 11, "bold"),
        "bg": "#34495e",
        "fg": "white",
        "width": 10,
        "bd": 0,
        "relief": "ridge",
        "activebackground": "#2c3e50",
        "activeforeground": "#f1c40f"
    }

    tk.Button(btn_frame, text="Add", **button_style).grid(row=0, column=0, padx=8)
    tk.Button(btn_frame, text="Update", **button_style).grid(row=0, column=1, padx=8)
    tk.Button(btn_frame, text="Delete", **button_style).grid(row=0, column=2, padx=8)
    tk.Button(btn_frame, text="Reset", **button_style).grid(row=0, column=3, padx=8)

    # Right Search & Display
    search_frame = tk.LabelFrame(customer_win, text="Search & Customer Records",
                                 font=("Segoe UI", 14, "bold"),
                                 bg="white", fg="#2c3e50")
    search_frame.place(x=580, y=100, width=600, height=500)

    search_and_display(search_frame)

    customer_win.mainloop()


def search_and_display(parent_frame):
    title_label = tk.Label(parent_frame, text="Search System",
                           font=("Segoe UI", 13, "bold"),
                           bg="white", fg="#2c3e50")
    title_label.grid(row=0, column=0, columnspan=6, pady=10)

    search_by_label = tk.Label(parent_frame, text="Search By:",
                               font=("Segoe UI", 11, "bold"),
                               fg="white", bg="#e74c3c")
    search_by_label.grid(row=1, column=0, padx=4, pady=5)

    search_by_options = ["Mobile", "Name", "Email", "Customer Ref"]
    search_by_var = tk.StringVar(value="Mobile")
    search_by_dropdown = ttk.Combobox(parent_frame, textvariable=search_by_var,
                                      values=search_by_options, state="readonly")
    search_by_dropdown.grid(row=1, column=1, padx=4)

    search_entry = tk.Entry(parent_frame, font=("Segoe UI", 11))
    search_entry.grid(row=1, column=2, padx=4)

    search_button = tk.Button(parent_frame, text="Search",
                              font=("Segoe UI", 11, "bold"),
                              bg="#2c3e50", fg="white", width=8)
    search_button.grid(row=1, column=3, padx=5)

    show_all_button = tk.Button(parent_frame, text="Show All",
                                font=("Segoe UI", 11, "bold"),
                                bg="#2c3e50", fg="white", width=8)
    show_all_button.grid(row=1, column=4, padx=5)

    # Treeview for customer data
    columns = ["Refer No", "Name", "Gender", "PostCode", "Mobile", "Email",
               "Nationality", "Id Proof Type", "Id Number", "Address"]

    tree = ttk.Treeview(parent_frame, columns=columns, show="headings", height=14)

    for col in columns:
        tree.heading(col, text=col)
        tree.column(col, width=100, anchor="center")

    tree.grid(row=2, column=0, columnspan=6, pady=20)


# Functions to Handle Windows
def fetch_customer_data():
    name_var.set("Kiran")
    gender_var.set("Male")

import tkinter as tk
from tkinter import ttk
from tkcalendar import DateEntry  # For date selection

def booking_window():
    booking_win = tk.Toplevel()
    booking_win.title("Room Booking Details")
    booking_win.geometry("1200x750")
    booking_win.configure(bg="#ecf0f1")  # Light background

    # Header Section
    header_frame = tk.Frame(booking_win, bg="#2c3e50", height=70)
    header_frame.pack(side="top", fill="x")

    title_label = tk.Label(header_frame, text="ROOM BOOKING DETAILS", 
                           font=("Segoe UI", 22, "bold"), fg="#f1c40f", bg="#2c3e50")
    title_label.pack(pady=15)

    # Left Frame - Booking Form
    form_frame = tk.LabelFrame(booking_win, text="Room Booking Form", 
                               font=("Segoe UI", 14, "bold"), bg="white", fg="#2c3e50")
    form_frame.place(x=20, y=100, width=550, height=530)

    tk.Label(form_frame, text="Customer Contact:", font=("Segoe UI", 11), bg="white").grid(row=0, column=0, padx=10, pady=8, sticky="w")
    contact_entry = tk.Entry(form_frame, font=("Segoe UI", 11), width=25)
    contact_entry.grid(row=0, column=1, padx=10, pady=8)

    fetch_button = tk.Button(form_frame, text="Fetch Data", font=("Segoe UI", 10, "bold"), 
                             bg="#34495e", fg="white", width=10)
    fetch_button.grid(row=0, column=2, padx=6)

    labels = ["Check-in Date:", "Check-out Date:", "Room Type:", "Available Room:", "Meal:", 
              "No Of Days:", "Paid Tax:", "Sub Total:", "Total Cost:"]
    entries = {}

    for i, label in enumerate(labels, start=1):
        lbl = tk.Label(form_frame, text=label, font=("Segoe UI", 11), bg="white")
        lbl.grid(row=i, column=0, padx=10, pady=5, sticky="w")

        if "Date" in label:
            entry = DateEntry(form_frame, font=("Segoe UI", 11), width=22, background='darkblue', foreground='white', borderwidth=2)
        elif label == "Room Type:":
            room_type_var = tk.StringVar(value="Single")
            entry = ttk.Combobox(form_frame, textvariable=room_type_var, values=["Deluxe", "Executive", "Suite"], state="readonly", width=20)
        else:
            entry = tk.Entry(form_frame, font=("Segoe UI", 11), width=25)

        entry.grid(row=i, column=1, padx=10, pady=5)
        entries[label] = entry

    bill_button = tk.Button(form_frame, text="Generate Bill", font=("Segoe UI", 11, "bold"),
                            bg="#16a085", fg="white", width=12)
    bill_button.grid(row=len(labels) + 1, column=0, columnspan=2, pady=10)

    # Right Frame - Customer Details
    details_frame = tk.LabelFrame(booking_win, text="Customer Details", 
                                  font=("Segoe UI", 14, "bold"), bg="white", fg="#2c3e50")
    details_frame.place(x=600, y=100, width=500, height=200)

    name_var = tk.StringVar()
    gender_var = tk.StringVar()

    tk.Label(details_frame, text="Name:", font=("Segoe UI", 11, "bold"), bg="white").grid(row=0, column=0, padx=10, pady=5, sticky="w")
    tk.Label(details_frame, textvariable=name_var, font=("Segoe UI", 11), bg="white").grid(row=0, column=1, padx=10, pady=5, sticky="w")

    tk.Label(details_frame, text="Gender:", font=("Segoe UI", 11, "bold"), bg="white").grid(row=1, column=0, padx=10, pady=5, sticky="w")
    tk.Label(details_frame, textvariable=gender_var, font=("Segoe UI", 11), bg="white").grid(row=1, column=1, padx=10, pady=5, sticky="w")

    # Buttons Section
    btn_frame = tk.Frame(booking_win, bg="#ecf0f1")
    btn_frame.place(x=40, y=650)

    for i, text in enumerate(["Add", "Update", "Delete", "Reset"]):
        tk.Button(btn_frame, text=text, font=("Segoe UI", 11, "bold"), bg="#34495e", fg="white", width=10).grid(row=0, column=i, padx=10)

    booking_win.mainloop()



def on_button_click(name):
    global dash
    if name == "CUSTOMER":
        add_customer_window()
    elif name == "BOOKING":
        booking_window()
    elif name == "ROOMS":
        room_management_window(dash)  # Pass the dash window as parent
    elif name == "LOGOUT":
        dash.destroy()
    else:
        print(f"{name} button clicked")

setup_db()
dashboard()
def signup_window():
    def register_user():
        username = user_entry.get()
        password = pass_entry.get()
        
        if username and password:
            conn = sqlite3.connect("hotel.db")
            cursor = conn.cursor()
            try:
                cursor.execute("INSERT INTO users (username, password) VALUES (?, ?)", (username, password))
                conn.commit()
                messagebox.showinfo("Success", "Registration Successful! Please Login.")
                reg.destroy()
            except sqlite3.IntegrityError:
                messagebox.showerror("Error", "Username already exists!")
            finally:
                conn.close()
        else:
            messagebox.showerror("Error", "All fields are required!")

    reg = tk.Toplevel()
    reg.title("Sign Up - Brightland Resorts and Spa")
    reg.geometry("400x300")

    tk.Label(reg, text="Username:", font=("Arial", 14)).pack(pady=5)
    user_entry = tk.Entry(reg, font=("Arial", 14))
    user_entry.pack(pady=5)

    tk.Label(reg, text="Password:", font=("Arial", 14)).pack(pady=5)
    pass_entry = tk.Entry(reg, show="*", font=("Arial", 14))
    pass_entry.pack(pady=5)

    tk.Button(reg, text="Sign Up", command=register_user, font=("Arial", 14), bg="green", fg="white").pack(pady=10)

def login_window():
    def check_login():
        username = user_entry.get()
        password = pass_entry.get()

        conn = sqlite3.connect("hotel.db")
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM users WHERE username=? AND password=?", (username, password))
        user = cursor.fetchone()
        conn.close()

        if user:
            messagebox.showinfo("Success", "Login Successful!")
            root.destroy()
            dashboard()
        else:
            messagebox.showerror("Error", "Invalid credentials")

    root = tk.Tk()
    root.title("Brightland Resorts and Spa - Staff Login")
    root.geometry("800x600")

    BG_PATH = r"C:\Rashi\College Assignments\SEM 4\MPR PYTHON\bg.png"
    LOGO_PATH = r"C:\Rashi\College Assignments\SEM 4\MPR PYTHON\brightland_logo.png.webp"

    # Load and Display Background
    if os.path.exists(BG_PATH):
        bg_image = Image.open(BG_PATH).resize((1300, 1300))
        bg_photo = ImageTk.PhotoImage(bg_image)
        bg_label = tk.Label(root, image=bg_photo)
        bg_label.image = bg_photo
        bg_label.place(x=0, y=0, relwidth=1, relheight=1)

    # Add Logo
    if os.path.exists(LOGO_PATH):
        logo_image = Image.open(LOGO_PATH).resize((150, 100))
        logo_photo = ImageTk.PhotoImage(logo_image)
        logo_label = tk.Label(root, image=logo_photo, bg='#ffffff')
        logo_label.image = logo_photo
        logo_label.place(relx=0.5, rely=0.1, anchor="center")

    frame = tk.Frame(root, bg="white")
    frame.place(relx=0.5, rely=0.5, anchor="center", width=320, height=320)
    tk.Label(frame, text="Welcome to Brightland Resorts & Spa ", font=("Arial", 13, "bold"), bg="white", fg="RED").pack(pady=5)

    tk.Label(frame, text="Staff Login", font=("Arial", 14, "bold"), bg="white", fg="GREEN").pack(pady=5)

    tk.Label(frame, text="Username:", font=("Arial", 12), bg="white").pack(pady=5)
    user_entry = tk.Entry(frame, font=("Arial", 12))
    user_entry.pack(pady=5)

    tk.Label(frame, text="Password:", font=("Arial", 12), bg="white").pack(pady=5)
    pass_entry = tk.Entry(frame, show="*", font=("Arial", 12))
    pass_entry.pack(pady=5)

    tk.Button(frame, text="Login", command=check_login, font=("Arial", 14), bg="#ff6600", fg="white").pack(pady=10)
    tk.Button(frame, text="Sign Up", command=signup_window, font=("Arial", 10), fg="blue", bg="white", relief="flat").pack(pady=5)

    root.mainloop()

if __name__ == "__main__":
    login_window()
   

def add_customer_window():
    customer_win = tk.Toplevel()
    customer_win.title("Add Customer Details")
    customer_win.geometry("1024x600")
    customer_win.configure(bg="white")

    # Header
    header_frame = tk.Frame(customer_win, bg="black", height=80)
    header_frame.pack(side="top", fill="x")

    title_label = tk.Label(header_frame, text="ADD CUSTOMER DETAILS", font=("Times New Roman", 24, "bold"), fg="gold", bg="black")
    title_label.pack(pady=20)

    # Form
    form_frame = tk.Frame(customer_win, bg="white")
    form_frame.place(x=20, y=100, width=480, height=480)

    labels = ["Customer Ref:", "Customer Name:", "Gender:", "PostCode:", "Mobile:", "Email:", "Nationality:", "Id Proof Type:", "Id Number:", "Address:"]
    entries = {}

    for i, label in enumerate(labels):
        lbl = tk.Label(form_frame, text=label, font=("Arial", 12), bg="white")
        lbl.grid(row=i, column=0, padx=10, pady=5, sticky="w")
        
        if label == "Gender:":  # Gender Dropdown
            gender_var = tk.StringVar(value="Male")
            gender_dropdown = ttk.Combobox(form_frame, textvariable=gender_var, values=["Male", "Female", "Other"], state="readonly")
            gender_dropdown.grid(row=i, column=1, padx=10, pady=5)
            entries[label] = gender_var
        elif label == "Nationality:":  # Nationality Dropdown
            nationality_var = tk.StringVar(value="Indian")
            nationality_dropdown = ttk.Combobox(form_frame, textvariable=nationality_var, values=["Indian", "American", "Other"], state="readonly")
            nationality_dropdown.grid(row=i, column=1, padx=10, pady=5)
            entries[label] = nationality_var
        elif label == "Id Proof Type:":  # ID Proof Dropdown
            proof_var = tk.StringVar(value="AdharCard")
            proof_dropdown = ttk.Combobox(form_frame, textvariable=proof_var, values=["AdharCard", "Passport", "Driving License"], state="readonly")
            proof_dropdown.grid(row=i, column=1, padx=10, pady=5)
            entries[label] = proof_var
        elif label == "Address:":  # Address Textbox
            entry = tk.Text(form_frame, width=20, height=4, font=("Arial", 12))
            entry.grid(row=i, column=1, padx=10, pady=5)
            entries[label] = entry
        else:  # Normal Entry
            entry = tk.Entry(form_frame, font=("Arial", 12))
            entry.grid(row=i, column=1, padx=10, pady=5)
            entries[label] = entry

    # Buttons
    btn_frame = tk.Frame(customer_win, bg="white")
    btn_frame.place(x=30, y=540)

    tk.Button(btn_frame, text="Add", font=("Arial", 12, "bold"), bg="black", fg="white", width=8).grid(row=0, column=0, padx=10)
    tk.Button(btn_frame, text="Update", font=("Arial", 12, "bold"), bg="black", fg="white", width=8).grid(row=0, column=1, padx=10)
    tk.Button(btn_frame, text="Delete", font=("Arial", 12, "bold"), bg="black", fg="white", width=8).grid(row=0, column=2, padx=10)
    tk.Button(btn_frame, text="Reset", font=("Arial", 12, "bold"), bg="black", fg="white", width=8).grid(row=0, column=3, padx=10)

# Dashboard Button Handling
def on_button_click(name):
    if name == "CUSTOMER":
        add_customer_window()
    elif name == "BOOKING":
        booking_window()
    else:
        print(f"{name} button clicked")
        

# Ensure dashboard's mainloop is only called once
dash.mainloop()
import tkinter as tk
from tkinter import ttk, messagebox
import sqlite3

class CustomerReport:
    def __init__(self, root):
        self.root = root
        self.root.title("Customer Report - Brightland Resorts and Spa")
        self.root.geometry("1000x550+200+100")
        self.root.config(bg="#fefefe")

        title = tk.Label(self.root, text="🧾 Customer Report", font=("Arial", 22, "bold"), bg="#283593", fg="white", pady=10)
        title.pack(fill=tk.X)

        # Search Frame
        search_frame = tk.Frame(self.root, bg="white", bd=2, relief=tk.RIDGE)
        search_frame.place(x=20, y=70, width=960, height=60)

        tk.Label(search_frame, text="Search by Name / Room No:", font=("Arial", 12), bg="white").place(x=10, y=18)
        self.search_var = tk.StringVar()
        search_entry = tk.Entry(search_frame, textvariable=self.search_var, font=("Arial", 12), width=25)
        search_entry.place(x=220, y=18)

        tk.Button(search_frame, text="Search", command=self.search_data, font=("Arial", 11), bg="#4CAF50", fg="white").place(x=470, y=15)
        tk.Button(search_frame, text="Show All", command=self.fetch_data, font=("Arial", 11), bg="#2196F3", fg="white").place(x=560, y=15)

        # Table Frame
        table_frame = tk.Frame(self.root, bd=3, relief=tk.RIDGE)
        table_frame.place(x=20, y=140, width=960, height=380)

        scroll_x = ttk.Scrollbar(table_frame, orient=tk.HORIZONTAL)
        scroll_y = ttk.Scrollbar(table_frame, orient=tk.VERTICAL)

        self.customer_table = ttk.Treeview(
            table_frame,
            columns=("ref", "name", "gender", "mobile", "email", "nationality", "idproof", "idnumber", "address", "checkin", "checkout", "roomtype", "roomno"),
            xscrollcommand=scroll_x.set,
            yscrollcommand=scroll_y.set
        )
        scroll_x.pack(side=tk.BOTTOM, fill=tk.X)
        scroll_y.pack(side=tk.RIGHT, fill=tk.Y)
        scroll_x.config(command=self.customer_table.xview)
        scroll_y.config(command=self.customer_table.yview)

        headings = ["Ref", "Name", "Gender", "Mobile", "Email", "Nationality", "ID Type", "ID No", "Address", "Check-In", "Check-Out", "Room Type", "Room No"]
        for i, col in enumerate(self.customer_table["columns"]):
            self.customer_table.heading(col, text=headings[i])
            self.customer_table.column(col, width=100, anchor=tk.CENTER)

        self.customer_table["show"] = "headings"
        self.customer_table.pack(fill=tk.BOTH, expand=1)

        self.fetch_data()

    def fetch_data(self):
        conn = sqlite3.connect("hotel.db")
        cur = conn.cursor()
        cur.execute("SELECT ref, name, gender, mobile, email, nationality, id_proof, id_number, address, check_in, check_out, room_type, room_no FROM customer")
        rows = cur.fetchall()
        self.customer_table.delete(*self.customer_table.get_children())
        for row in rows:
            self.customer_table.insert("", tk.END, values=row)
        conn.close()

    def search_data(self):
        key = self.search_var.get()
        conn = sqlite3.connect("hotel.db")
        cur = conn.cursor()
        cur.execute("SELECT ref, name, gender, mobile, email, nationality, id_proof, id_number, address, check_in, check_out, room_type, room_no FROM customer WHERE name LIKE ? OR room_no LIKE ?", ('%'+key+'%', '%'+key+'%'))
        rows = cur.fetchall()
        self.customer_table.delete(*self.customer_table.get_children())
        if rows:
            for row in rows:
                self.customer_table.insert("", tk.END, values=row)
        else:
            messagebox.showinfo("Not Found", "No matching records found.")
        conn.close()

# Run this window standalone
if __name__ == "__main__":
    root = tk.Tk()
    app = CustomerReport(root)
    root.mainloop()
import tkinter as tk
from tkinter import ttk, messagebox
import sqlite3

class CustomerReport:
    def __init__(self, root):
        self.root = root
        self.root.title("Customer Report - Brightland Resorts and Spa")
        self.root.geometry("1000x550+200+100")
        self.root.config(bg="#fefefe")

        title = tk.Label(self.root, text="🧾 Customer Report", font=("Arial", 22, "bold"), bg="#283593", fg="white", pady=10)
        title.pack(fill=tk.X)

        # Search Frame
        search_frame = tk.Frame(self.root, bg="white", bd=2, relief=tk.RIDGE)
        search_frame.place(x=20, y=70, width=960, height=60)

        tk.Label(search_frame, text="Search by Name / Room No:", font=("Arial", 12), bg="white").place(x=10, y=18)
        self.search_var = tk.StringVar()
        search_entry = tk.Entry(search_frame, textvariable=self.search_var, font=("Arial", 12), width=25)
        search_entry.place(x=220, y=18)

        tk.Button(search_frame, text="Search", command=self.search_data, font=("Arial", 11), bg="#4CAF50", fg="white").place(x=470, y=15)
        tk.Button(search_frame, text="Show All", command=self.fetch_data, font=("Arial", 11), bg="#2196F3", fg="white").place(x=560, y=15)

        # Table Frame
        table_frame = tk.Frame(self.root, bd=3, relief=tk.RIDGE)
        table_frame.place(x=20, y=140, width=960, height=380)

        scroll_x = ttk.Scrollbar(table_frame, orient=tk.HORIZONTAL)
        scroll_y = ttk.Scrollbar(table_frame, orient=tk.VERTICAL)

        self.customer_table = ttk.Treeview(
            table_frame,
            columns=("ref", "name", "gender", "mobile", "email", "nationality", "idproof", "idnumber", "address", "checkin", "checkout", "roomtype", "roomno"),
            xscrollcommand=scroll_x.set,
            yscrollcommand=scroll_y.set
        )
        scroll_x.pack(side=tk.BOTTOM, fill=tk.X)
        scroll_y.pack(side=tk.RIGHT, fill=tk.Y)
        scroll_x.config(command=self.customer_table.xview)
        scroll_y.config(command=self.customer_table.yview)

        headings = ["Ref", "Name", "Gender", "Mobile", "Email", "Nationality", "ID Type", "ID No", "Address", "Check-In", "Check-Out", "Room Type", "Room No"]
        for i, col in enumerate(self.customer_table["columns"]):
            self.customer_table.heading(col, text=headings[i])
            self.customer_table.column(col, width=100, anchor=tk.CENTER)

        self.customer_table["show"] = "headings"
        self.customer_table.pack(fill=tk.BOTH, expand=1)

        self.fetch_data()

    def fetch_data(self):
        conn = sqlite3.connect("hotel.db")
        cur = conn.cursor()
        cur.execute("SELECT ref, name, gender, mobile, email, nationality, id_proof, id_number, address, check_in, check_out, room_type, room_no FROM customer")
        rows = cur.fetchall()
        self.customer_table.delete(*self.customer_table.get_children())
        for row in rows:
            self.customer_table.insert("", tk.END, values=row)
        conn.close()

    def search_data(self):
        key = self.search_var.get()
        conn = sqlite3.connect("hotel.db")
        cur = conn.cursor()
        cur.execute("SELECT ref, name, gender, mobile, email, nationality, id_proof, id_number, address, check_in, check_out, room_type, room_no FROM customer WHERE name LIKE ? OR room_no LIKE ?", ('%'+key+'%', '%'+key+'%'))
        rows = cur.fetchall()
        self.customer_table.delete(*self.customer_table.get_children())
        if rows:
            for row in rows:
                self.customer_table.insert("", tk.END, values=row)
        else:
            messagebox.showinfo("Not Found", "No matching records found.")
        conn.close()

# Run this window standalone
if __name__ == "__main__":
    root = tk.Tk()
    app = CustomerReport(root)
    root.mainloop()


 
# Run Database Setup and Login
setup_db()
login_window()


