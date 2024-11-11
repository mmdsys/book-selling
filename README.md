import sqlite3  
from tkinter import *  
from tkinter import messagebox  
from tkinter import simpledialog

#==========================================================MMDSYs==================================================

class Bookstore:  
    def __init__(self):  
        self.books = []  
        self.total_sales = 0  
        self.load_books_from_db()  

    def load_books_from_db(self):  
        conn = sqlite3.connect('books.db')  
        cur = conn.cursor()  
        cur.execute("CREATE TABLE IF NOT EXISTS books (id INTEGER PRIMARY KEY, name TEXT, author TEXT, price REAL, isbn TEXT, number INTEGER)")  
        conn.commit()  
        cur.execute("SELECT * FROM books")  
        self.books = cur.fetchall()  
        conn.close()  

    def add_book(self, name, author, price, isbn, number):  
        conn = sqlite3.connect('books.db')  
        cur = conn.cursor()  
        cur.execute("INSERT INTO books (name, author, price, isbn, number) VALUES (?, ?, ?, ?, ?)", (name, author, price, isbn, number))  
        conn.commit()  
        conn.close()  
        self.load_books_from_db()   

    def sell_book(self, name, author, number):  
        self.load_books_from_db() 
        for book in self.books:  
            if book[1] == name and book[2] == author:  
                if book[5] < number:  
                    return "Not enough number of books available."  
                total_price = number * book[3]  
                self.total_sales += total_price  

                
                new_number = book[5] - number  
                self.update_book_number(book[0], new_number)     
                return f"Total Price: ${total_price:.2f}"  
        return "Book not found."  
          

    def update_book_number(self, book_id, new_number):  
        conn = sqlite3.connect('books.db')  
        cur = conn.cursor()  
        cur.execute("UPDATE books SET number=? WHERE id=?", (new_number, book_id))  
        conn.commit()  
        conn.close()  

    def remove_book(self, book_id):  
        conn = sqlite3.connect('books.db')  
        cur = conn.cursor()  
        cur.execute("DELETE FROM books WHERE id=?", (book_id,))  
        conn.commit()  
        conn.close()  
        self.load_books_from_db()  

    def search_book(self, name, author):  
        result = []  
        for book in self.books: 
            #میشد به حروف کوچک تبدیل نکرد اما برای افزایش دقت سرچ این کار رو انجام دادم  
            if (name.lower() in book[1].lower() and author.lower() in book[2].lower()):  
                result.append(book)  
        return result  

    def sales_report(self):  
        return f"Total Sales: ${self.total_sales:.2f}"  

class BookstoreApp:  
    def __init__(self, root, bookstore):  
        self.root = root  
        self.bookstore = bookstore  
        self.create_widgets()  

    def create_widgets(self):  
        self.root.title("سامانه فروش کتاب")  
        self.root.attributes('-fullscreen', True)  
        self.root.bind('<Escape>', lambda a: self.root.attributes('-fullscreen', False))  

        self.name_text   = StringVar()  
        self.author_text = StringVar()  
        self.price_text  = StringVar()  
        self.isbn_text   = StringVar()  
        self.number_text = StringVar()  
        
        l1 = Label(self.root, text="Name", font=("Arial", 24))  
        l1.grid(row=0, column=0, padx=10, pady=10, sticky=W)  
        
        l2 = Label(self.root, text="Author", font=("Arial", 24))  
        l2.grid(row=0, column=2, padx=10, pady=10, sticky=W)  
        
        l3 = Label(self.root, text="Price", font=("Arial", 24))  
        l3.grid(row=1, column=0, padx=10, pady=10, sticky=W)  
        
        l4 = Label(self.root, text="ISBN", font=("Arial", 24))  
        l4.grid(row=1, column=2, padx=10, pady=10, sticky=W)  
        
        l5 = Label(self.root, text="Number", font=("Arial", 24))  
        l5.grid(row=2, column=0, padx=10, pady=10, sticky=W)  

        e1 = Entry(self.root, textvariable=self.name_text, font=("Arial", 24), width=20)  
        e1.grid(row=0, column=1, padx=10, pady=10, sticky=W+E)  
        
        e2 = Entry(self.root, textvariable=self.author_text, font=("Arial", 24), width=20)  
        e2.grid(row=0, column=3, padx=10, pady=10, sticky=W+E)  

        e3 = Entry(self.root, textvariable=self.price_text, font=("Arial", 24), width=20)  
        e3.grid(row=1, column=1, padx=10, pady=10, sticky=W+E)  

        e4 = Entry(self.root, textvariable=self.isbn_text, font=("Arial", 24), width=20)  
        e4.grid(row=1, column=3, padx=10, pady=10, sticky=W+E)  

        e5 = Entry(self.root, textvariable=self.number_text, font=("Arial", 24), width=20)  
        e5.grid(row=2, column=1, padx=10, pady=10, sticky=W+E)  
            
        self.list_frame = Frame(self.root)  
        self.list_frame.grid(row=3, column=0, columnspan=2, rowspan=4, padx=10, pady=10, sticky=NSEW)  

        self.list1 = Listbox(self.list_frame, font=("Arial", 16), width=30)  
        self.list1.pack(side=LEFT, fill=BOTH, expand=True)  

        sb1 = Scrollbar(self.list_frame)  
        sb1.pack(side=RIGHT, fill=Y)  

        self.list1.configure(yscrollcommand=sb1.set)  
        sb1.configure(command=self.list1.yview)    

        button_frame = Frame(self.root)  
        button_frame.grid(row=3, column=3, rowspan=4, padx=10, pady=10)  

    
        Button(button_frame, text="List all books", width=20, font=("Arial", 24), bg="gold",command=self.list_books).grid(row=0, column=0, pady=10, padx=10)
        Button(button_frame, text="Add Book" , width=20, font=("Arial", 24), bg="gold", command=self.add_book).grid(row=1, column=0, pady=10, padx=10)  
        Button(button_frame, text="Sell Book"   , width=20, font=("Arial", 24), bg="gold", command=self.sell_book).grid(row=2, column=0, pady=10, padx=10)  
        Button(button_frame, text="Search Book"   , width=20, font=("Arial", 24), bg="gold",command=self.search_book).grid(row=3, column=0, pady=10, padx=10)  
        Button(button_frame, text="Remove a Book" , width=20, font=("Arial", 24), bg="gold", command=self.remove_book).grid(row=4, column=0, pady=10, padx=10)  
        Button(button_frame, text="Sales Report" , width=20, font=("Arial", 24), bg="gold", command=self.show_sales_report).grid(row=5, column=0, pady=10, padx=10)  
        Button(button_frame, text="Exit"    , width=20, font=("Arial", 24), bg="red" ,command=self.root.quit).grid(row=6, column=0, pady=10, padx=10)
    
    def list_books(self):  
        self.list1.delete(0, END)  
        for book in self.bookstore.books:  
            self.list1.insert(END, f"{book[1]} by {book[2]} - Price: ${book[3]:.2f} - Number: {book[5]}")  
    

    
    def add_book(self):  
        name = self.name_text.get()  
        author = self.author_text.get()  
        price = float(self.price_text.get())  
        isbn = self.isbn_text.get()  
        number = int(self.number_text.get())  
        self.bookstore.add_book(name, author, price, isbn, number)  
        self.list_books()    
        messagebox.showinfo("Success", "Book added successfully")  

    def sell_book(self):  
        selected_book = self.list1.curselection()  
        if not selected_book:  
            messagebox.showwarning("Warning", "PlZ select a book to sell!")  
            return  

        book_info = self.list1.get(selected_book)  
        selected_book_name = book_info.split(" by ")[0]  
        selected_book_author = book_info.split(" - ")[0].split(" by ")[1]  

        number_to_sell = simpledialog.askinteger("Input", "Enter number of books to sell:", minvalue=1)  
        if number_to_sell is not None:  
            result = self.bookstore.sell_book(selected_book_name, selected_book_author, number_to_sell)  
        
            messagebox.showinfo("Result", result)  
        
        self.list_books()

    def search_book(self):  
        name = self.name_text.get()  
        author = self.author_text.get()  
        results = self.bookstore.search_book(name, author)  
        self.list1.delete(0, END) 
        for book in results:  
            self.list1.insert(END, f"{book[1]} by {book[2]} - Price: ${book[3]:.2f} - Number: {book[5]}")    
        if not results:  
            messagebox.showinfo("Search", "No books found.")  

    def remove_book(self):  
        selected_book = self.list1.curselection()  
        if not selected_book:  
            messagebox.showwarning("Warning", "PlZ select a book to remove")  
            return  
        
        book_info = self.list1.get(selected_book)  
        book_id = self.bookstore.books[selected_book[0]][0]   
        self.bookstore.remove_book(book_id)  
        self.list_books()  
        messagebox.showinfo("Success", "Book removed successfully")  

    def show_sales_report(self):  
        report = self.bookstore.sales_report()  
        messagebox.showinfo("Sales Report", report)  

if __name__ == '__main__':  
    root = Tk()  
    bookstore = Bookstore()  
    app = BookstoreApp(root, bookstore)  
    root.mainloop()
