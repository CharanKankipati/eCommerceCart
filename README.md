import tkinter as tk
from tkinter import messagebox
from tkinter import ttk
from tkinter import *
from tkinter.ttk import *
import matplotlib.pyplot as plt
import pandas as pd


class BasePage:
    def __init__(self, master):
        self.master = master

    def show(self):
        self.master.mainloop()

    def hide(self):
        self.master.withdraw()


class LoginPage(BasePage):
    def __init__(self, master):
        super().__init__(master)
        master.title("Login")
        master.configure(bg='#68e0f2')

        self.username_label = tk.Label(master, text="Username:", bg='#68e0f2', font=("Arial", 12))
        self.username_label.pack()

        self.username_entry = tk.Entry(master, font=("Arial", 12))
        self.username_entry.pack()

        self.password_label = tk.Label(master, text="Password:", bg='#68e0f2', font=("Arial", 12))
        self.password_label.pack()

        self.password_entry = tk.Entry(master, show="*", font=("Arial", 12))
        self.password_entry.pack()

        self.login_button_style = ttk.Style()
        self.login_button_style.configure('Login.TButton', background='blue', foreground='black')
        self.login_button = ttk.Button(master, text="Login", command=self.login, style='Login.TButton')
        self.login_button.pack(pady=10)

    def login(self):
        username = self.username_entry.get()
        password = self.password_entry.get()

        if username == "oops" and password == "123456":
            self.master.destroy()
            root = tk.Tk()
            app = ShoppingCartApp(root)
            app.show()
        else:
            messagebox.showerror("Login Failed", "Invalid username or password")


class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price

    def __str__(self):
        return f"{self.name} ({self.price}/-)"


class CartItem:
    def __init__(self, product, quantity=1):
        self.product = product
        self.quantity = quantity

    def increment_quantity(self, amount=1):
        self.quantity += amount

    def decrement_quantity(self, amount=1):
        if self.quantity > 0:
            self.quantity -= amount
        if self.quantity < 1:
            self.quantity = 1

    def total_price(self):
        return self.product.price * self.quantity

    def __str__(self):
        return f"{self.product.name} - {self.quantity} item(s) - {self.total_price()}/-"


class ShoppingCart:
    def __init__(self):
        self.items = []

    def add_product(self, product, quantity=1):
        for item in self.items:
            if item.product.name == product.name:
                item.increment_quantity(quantity)
                return
        self.items.append(CartItem(product, quantity))

    def remove_product(self, product_name):
        for item in self.items:
            if item.product.name == product_name:
                self.items.remove(item)
                return

    def update_quantity(self, product_name, quantity):
        for item in self.items:
            if item.product.name == product_name:
                item.quantity = quantity
                if item.quantity < 1:
                    self.items.remove(item)
                return

    def total_cost(self):
        return sum(item.total_price() for item in self.items)

    def total_items(self):
        return sum(item.quantity for item in self.items)

    def get_cart_summary(self):
        cart_summary = "\n".join(str(item) for item in self.items)
        cart_summary += f"\nTotal Price: {self.total_cost()}/-"
        return cart_summary


class ShoppingCartApp(BasePage):
    def __init__(self, master):
        super().__init__(master)
        master.title("Shopping Cart")
        master.configure(bg='#68e0f2')
        menubar = Menu(master)

        file = Menu(menubar, tearoff=0)
        menubar.add_cascade(label='Menu', menu=file)
        file.add_command(label='Your Orders', command=None)
        file.add_command(label='Your Account', command=None)
        file.add_command(label='Sign Out', command=None)
        file.add_command(label='Customer Service', command=None)
        file.add_separator()
        file.add_command(label='Exit', command=master.destroy)

        edit = Menu(menubar, tearoff=0)
        menubar.add_cascade(label='Account', menu=edit)
        edit.add_command(label='Login & security', command=None)
        edit.add_command(label='Your Addresses', command=None)
        edit.add_command(label='Profile', command=None)
        edit.add_separator()
        edit.add_command(label='About', command=None)
        edit.add_command(label='Your Recommendations', command=None)

        help_ = Menu(menubar, tearoff=0)
        menubar.add_cascade(label='Help', menu=help_)
        help_.add_command(label='Any Queries', command=None)
        help_.add_separator()
        help_.add_command(label='Contact:444-5656-123', command=None)

        sales = Menu(menubar, tearoff=0)
        menubar.add_cascade(label='Sales', menu=sales)
        sales.add_command(label='Show Sales Graph', command=self.show_sales_graph)

        master.config(menu=menubar)

        self.cart = ShoppingCart()
        self.products_df = pd.DataFrame({
            'Name': ['AC', 'TV', 'Washing Machine', 'cooler', 'tablet', 'smart home alexa', 'sub whoofer', 'amplifier', 'induction cooktop', 'airfryer', 'gas stove', 'tea powder-1kg',
                     'soft drink-750ml', 'cheese', 'paneer', 'brown bread', 'rice-1kg', 'wheat flour-1kg', 'sugar-1kg', 'salt-1kg'],
            'Price': [45000, 50000, 70000, 30000, 17999, 6599, 6550, 3399, 8500, 12500, 2500, 450, 45, 120, 180, 45, 80, 30, 50, 30]
        })
        self.products = [Product(row['Name'], row['Price']) for index, row in self.products_df.iterrows()]

        self.label = tk.Label(master, text="Add products to the shopping cart:", bg='#68e0f2', font=("Arial", 12))
        self.label.pack()

        self.listbox = tk.Listbox(master, height=20, width=100, bg='white', fg='black', font=("Arial", 12))
        self.listbox.pack()
        for product in self.products:
            self.listbox.insert(tk.END, str(product))

        self.quantity_label = tk.Label(master, text="Quantity:", bg='#68e0f2', font=("Arial", 12))
        self.quantity_label.pack()

        self.quantity_entry = tk.Entry(master, font=("Arial", 12))
        self.quantity_entry.pack()

        self.add_button_style = ttk.Style()
        self.add_button_style.configure('Add.TButton', background='green', foreground='black')
        self.add_button = ttk.Button(master, text="Add to Cart", command=self.add_to_cart, style='Add.TButton')
        self.add_button.pack()

        self.cart_contents = tk.Text(master, height=10, width=50, bg='white', fg='black', font=("Arial", 12))
        self.cart_contents.pack()

        self.remove_button_style = ttk.Style()
        self.remove_button_style.configure('Remove.TButton', background='red', foreground='black')
        self.remove_button = ttk.Button(master, text="Remove Selected", command=self.remove_from_cart, style='Remove.TButton')
        self.remove_button.pack()

        def open_new_window():
            total_products = self.cart.total_items()
            total_price = self.cart.total_cost()
            new_window = tk.Toplevel(master)
            new_window.title("CART")
            new_window.geometry("200x200")
            label = tk.Label(new_window, text=f"Total Products: {total_products}\nTotal Price: {total_price}/-\nOrder Placed!\nThank you for visiting!", font=("Arial", 12))
            label.pack()

        self.buy_button_style = ttk.Style()
        self.buy_button_style.configure('Buy.TButton', background='blue', foreground='black')
        btn = ttk.Button(master, text="Buy now", command=open_new_window, style='Buy.TButton')
        btn.pack(pady=10)

    def add_to_cart(self):
        product_index = self.listbox.curselection()[0]
        product = self.products[product_index]
        try:
            quantity = int(self.quantity_entry.get())
            if quantity > 0:
                self.cart.add_product(product, quantity)
                self.update_cart_display()
        except ValueError:
            messagebox.showerror("Invalid Input", "Please enter a valid quantity")

    def remove_from_cart(self):
        product_index = self.listbox.curselection()[0]
        product = self.products[product_index]
        self.cart.remove_product(product.name)
        self.update_cart_display()

    def update_cart_display(self):
        self.cart_contents.delete(1.0, tk.END)
        self.cart_contents.insert(tk.END, self.cart.get_cart_summary())

    def show_sales_graph(self):
        fig, ax = plt.subplots()
        x = [2019, 2020, 2021, 2022, 2023, 2024]
        y = [4.0, 5.7, 9.0, 7.7, 9.4, 11.1]
        sizes = [30 * k for k in y]
        ax.scatter(x, y, label="Annual income", s=sizes, facecolor="C0", edgecolor='k')
        ax.set_title("INCOME OVER YEARS")
        ax.set_ylabel('INCOME IN LAKHS')
        ax.set_xlabel("YEAR")
        ax.legend()
        plt.show()


if __name__ == "__main__":
    root = tk.Tk()
    login_page = LoginPage(root)
    login_page.show()
