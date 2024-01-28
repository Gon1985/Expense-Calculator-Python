import tkinter as tk
from tkinter import ttk, messagebox
from PIL import Image, ImageTk
import os
import datetime
import pygame
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
from tkinter import PhotoImage


class ExpenseCalculatorGUI:
    def __init__(self):
        self.root = tk.Tk()
        self.root.title("Expense Calculator")
        self.root.geometry("400x500")
        self.root.resizable(False, False)

        
        icon_path = os.path.join("images", "icon.ico")
        self.root.iconbitmap(icon_path)

        
        pygame.mixer.init()

        
        background_image = Image.open("images/background01.png")
        self.background_photo = ImageTk.PhotoImage(background_image)

        
        self.canvas = tk.Canvas(self.root, width=400, height=500)
        self.canvas.pack()

        
        self.canvas.create_image(0, 0, anchor=tk.NW, image=self.background_photo)

        
        self.canvas.configure(bg="#2629de")

        
        self.expenses = []

        button_font = ("Arial", 12)

        
        self.add_sound = pygame.mixer.Sound("sounds/sound2.wav")
        self.show_sound = pygame.mixer.Sound("sounds/sound2.wav")
        self.calculate_sound = pygame.mixer.Sound("sounds/sound3.wav")
        self.pie_chart_sound = pygame.mixer.Sound("sounds/sound2.wav") 
        self.exit_sound = pygame.mixer.Sound("sounds/sound1.wav")

        
        self.description_label = tk.Label(self.root, text="Expense Description:")
        self.description_entry = tk.Entry(self.root)
        self.amount_label = tk.Label(self.root, text="Expense Amount:")
        self.amount_entry = tk.Entry(self.root)

        
        self.currency_label = tk.Label(self.root, text="Currency:")
        self.currency_var = tk.StringVar()
        self.currency_combobox = ttk.Combobox(self.root, textvariable=self.currency_var, values=["USD", "GBP", "EUR", "ARS", "BRL", "VND", "JPY", "UYI", "RUB", "KRW", "KPW", "CNY", "MXN"], state="readonly")
        self.currency_combobox.set("USD")  

        self.add_button = tk.Button(self.root, text="Add Expense", command=self.add_expense, bg="#fc0303", fg="white", font=button_font)
        self.show_button = tk.Button(self.root, text="Show Expenses", command=self.show_expenses, bg="#2196f3", fg="white", font=button_font)
        self.calculate_button = tk.Button(self.root, text="Calculate Total", command=self.calculate_total, bg="#3cc700", fg="white", font=button_font)
        
        self.pie_chart_button = tk.Button(self.root, text="Pie Chart", command=self.show_pie_chart, bg="#ff9800", fg="white", font=button_font)
        self.exit_button = tk.Button(self.root, text="Exit", command=self.confirm_exit, bg="#607d8b", fg="white", font=button_font)

        
        self.description_label.place(x=20, y=20)
        self.description_entry.place(x=200, y=20)
        self.amount_label.place(x=20, y=60)
        self.amount_entry.place(x=200, y=60)
        self.currency_label.place(x=20, y=100)
        self.currency_combobox.place(x=200, y=100)
        self.add_button.place(x=20, y=140, width=360)
        self.show_button.place(x=20, y=180, width=360)
        self.calculate_button.place(x=20, y=220, width=360)
        self.pie_chart_button.place(x=20, y=260, width=360)  
        self.exit_button.place(x=20, y=300, width=360)

        
        self.root.bind("<Button-3>", lambda event: self.show_popup_menu(event))
        
        

        
        self.root.popup_menu = tk.Menu(self.root, tearoff=0)
        self.root.popup_menu.add_command(label="About", command=self.show_about_window)
        self.root.popup_menu.add_command(label="Help", command=self.show_help_window)
        self.root.popup_menu.add_command(label="Save Expenses", command=self.save_expenses_to_file)

        self.pie_chart_window_open = False
        
        
        
    def save_expenses_to_file(self):
        try:
            with open("expenses.txt", "w") as file:
                for expense in self.expenses:
                    total_expenses = sum(exp['amount'] for exp in self.expenses)
                    file.write(f"{expense['description']} | {expense['amount']} | {expense['currency']} | Total: ${total_expenses} | {expense['date']}\n")
            messagebox.showinfo("Save Successful", "Expenses information saved successfully.")
        except Exception as e:
            messagebox.showerror("Error", f"Error saving expenses information: {e}")

        
        
    def show_pie_chart(self):
        if self.expenses:
            labels = [expense["description"] for expense in self.expenses]
            amounts = [expense["amount"] for expense in self.expenses]

            
            if not self.pie_chart_window_open:
                
                self.pie_chart_window_open = True

                
                pie_chart_window = tk.Toplevel(self.root)
                pie_chart_window.title("Expense Pie Chart")
                pie_chart_window.resizable(False, False)

                
                icon_path = os.path.join("images", "icon.ico")  
                pie_chart_window.iconbitmap(icon_path)

                
                pie_chart_window.configure(bg="#2629de")  

                
                fig, ax = plt.subplots()

                
                colors = ["#ff0000", "#ffd900", "#3fdb0f", "#d817ff", "#38fcff", "#88f03a", "#4587f7", "#ff19dd", "#ffbb00", "#6600ff", "#a3e300", "#ff0051", "#af52c4" ]  # Replace with your preferred colors
                wedges, text, autotext = ax.pie(amounts, labels=labels, autopct='%1.1f%%', startangle=90, colors=colors)

                
                ax.axis('equal')

                
                canvas = FigureCanvasTkAgg(fig, master=pie_chart_window)
                canvas_widget = canvas.get_tk_widget()
                canvas_widget.pack()

                def on_close():
                    
                    nonlocal self
                    self.pie_chart_window_open = False
                    pie_chart_window.destroy()

                
                pie_chart_window.protocol("WM_DELETE_WINDOW", on_close)

                
                pie_chart_window.update_idletasks()

                
                pie_chart_window.mainloop()
            else:
                messagebox.showinfo("Info", "Expense Pie Chart is already open.")
        else:
            messagebox.showinfo("Info", "No expenses recorded.")

    def play_sound(self, sound):
        pygame.mixer.Sound.play(sound)

    def add_expense(self):
        self.play_sound(self.add_sound)
        description = self.description_entry.get()
        amount_str = self.amount_entry.get()
        currency = self.currency_var.get()

        if description and amount_str and currency:
            try:
                amount = float(amount_str)
                self.expenses.append({"description": description, "amount": amount, "currency": currency, "date": datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")})
                messagebox.showinfo("Expense Added", "Expense added successfully!")

                
                self.update_monthly_totals()

            except ValueError:
                messagebox.showerror("Error", "Invalid amount. Please enter a valid number.")
        else:
            messagebox.showerror("Error", "Description, amount, and currency are required.")

    def show_expenses(self):
        self.play_sound(self.show_sound)
        if self.expenses:
            expense_list = "\n".join([f"{expense['description']}: ${expense['amount']} ({expense['currency']})" for expense in self.expenses])
            messagebox.showinfo("Expense List", expense_list)
        else:
            messagebox.showinfo("Expense List", "No expenses recorded.")

    def calculate_total(self):
        self.play_sound(self.calculate_sound)
        if self.expenses:
            total = sum(expense["amount"] for expense in self.expenses)
            messagebox.showinfo("Total Expenses", f"Total Expenses: ${total}")
        else:
            messagebox.showinfo("Total Expenses", "No expenses recorded.")

    
    def show_about_window(self):
        if not self.is_window_open("About"):
            about_window = tk.Toplevel(self.root)
            about_window.title("About")
            about_window.geometry("510x100")
            about_window.resizable(False, False)

            
            icon_path = os.path.join("images", "icon.ico")  
            about_window.iconbitmap(icon_path)

            
            image_path = os.path.join("images", "background2.png")  
            background_image = Image.open(image_path)
            background_photo = ImageTk.PhotoImage(background_image)

            
            img_width, img_height = background_image.size

            
            canvas = tk.Canvas(about_window, width=img_width, height=img_height)
            canvas.pack()

            canvas.create_image(0, 0, anchor=tk.NW, image=background_photo)

            about_text = "EG 1.0 / This program was created by Gonzalo Ezequiel Temes in 2024 / Kunker Studios RD."

            about_message = tk.Message(canvas, text=about_text, bg="#2629de", fg="white", font=("Arial", 10), width=img_width-40)
            about_message.place(relx=0.5, rely=0.5, anchor=tk.CENTER)

            
            canvas.image = background_photo

    def is_window_open(self, window_title):
        open_windows = self.root.winfo_children()
        for child in open_windows:
            if isinstance(child, tk.Toplevel) and child.title() == window_title:
                return True
        return False
        
        
    def show_help_window(self):
        if not self.is_window_open("Help"):
            help_window = tk.Toplevel(self.root)
            help_window.title("Help")
            help_window.geometry("400x500")
            help_window.resizable(False, False)

            
            icon_path = os.path.join("images", "icon.ico")  
            help_window.iconbitmap(icon_path)

            
            help_window.configure(bg="#2629de")

            
            image_path = os.path.join("images", "background3.png")  
            background_image = Image.open(image_path)
            background_photo = ImageTk.PhotoImage(background_image)

            
            img_width, img_height = background_image.size

            
            canvas = tk.Canvas(help_window, width=img_width, height=img_height)
            canvas.pack()

            canvas.create_image(0, 0, anchor=tk.NW, image=background_photo)

            

            help_text = "Welcome! This is the help section.\n\n" \
                "1) First, enter a brief description in the first box.\n\n" \
                "2) Second, add the amount\n\n" \
                "3) Third, choose the correct country currency\n\n" \

            help_message = tk.Message(canvas, text=help_text, bg="#2629de", fg="white", font=("Arial", 10), width=img_width-40)
            help_message.place(relx=0.5, rely=0.5, anchor=tk.CENTER)

            
            canvas.image = background_photo

    def is_window_open(self, window_title):
        open_windows = self.root.winfo_children()
        for child in open_windows:
            if isinstance(child, tk.Toplevel) and child.title() == window_title:
                return True
        return False
   
    def show_popup_menu(self, event):
        self.root.popup_menu.post(event.x_root, event.y_root)
     

    def update_monthly_totals(self):
        
        self.monthly_totals = {}
        for expense in self.expenses:
            date = datetime.datetime.strptime(expense["date"], "%Y-%m-%d %H:%M:%S")
            month_year = date.strftime("%B %Y")
            if month_year not in self.monthly_totals:
                self.monthly_totals[month_year] = {"total": 0, "date": date}
            self.monthly_totals[month_year]["total"] += expense["amount"]
            
            
    def confirm_exit(self):
        self.play_sound(self.exit_sound)  
        if messagebox.askokcancel("Confirm Exit", "Are you sure you want to exit?"):
            self.root.destroy() 
            
            

    def run_gui(self):
        self.root.mainloop()


if __name__ == "__main__":
    expense_calculator_gui = ExpenseCalculatorGUI()
    expense_calculator_gui.run_gui()
