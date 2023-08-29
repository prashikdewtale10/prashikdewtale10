### Hi there 👋 this is Prashik Dewtale


#!/usr/bin/env python3
"""
main.py
-------------
Script to manage book inventary.

@author: prashikdewtale
"""


import math                             # for rounding book price to 2 decimal points
import csv                              # for working with csv file
import os                               # for directory related operations
from datetime import datetime           # for showing date and time 
# import logging                        # for logging related tasks

# # Configuring basicConfig() functio
# logging.basicConfig(filename='errors.log',filemode='w',format='%(asctime)s - %(levelname)s - %(message)s')


class Book:

    def __init__(self,book_id,book_title,book_author,book_price,book_count,book_date):

        self.book_id = book_id
        self.book_title = book_title
        self.book_author = book_author
        self.book_price = book_price
        self.book_count = book_count
        self.book_date = book_date


class Inventary:

    def __init__(self):

        self.books = []
        self.file_name = 'books.csv'


    def add_book(self):

        """
        Taking Inputs from user and writing data to file
        """

        try:   # to handle valueError Exceptions.

            now = datetime.now()
            book_id = input("Enter Book Id: ")
            book_title = input("Enter Book Title: ")
            book_author = input("Enter Book Author: ")
            book_price = float(input("Enter Book Price: "))
            book_count = int(input("Enter Book Available: "))
            book_date =  now.strftime("%d/%m/%Y %H:%M:%S")

            book = Book(book_id,book_title,book_author,book_price,book_count,book_date)
            self.books.append(book)
            self.save_books
            ()
            self.books = []

        except ValueError:
            # logging.warning('Invalid type input given')
            print("Book Not Added, Enter Value with specified type")
            print()



    def save_books(self):

        """
        Method Reads the data from Inevntary class Books List and write data into csv file
        """

        files = os.listdir()

        if self.file_name in files:
            with open(self.file_name,'a',newline='') as fp:
                writer = csv.writer(fp)
                for book in self.books:
                    writer.writerow([book.book_id,book.book_title,book.book_author,book.book_price,book.book_count,book.book_date])     

        else:
            with open(self.file_name,'w',newline='') as fp:
                writer = csv.writer(fp)
                for book in self.books:
                    writer.writerow([book.book_id,book.book_title,book.book_author,book.book_price,book.book_count,book.book_date])




    def list_books(self):

        """
        Displays the Books available in file
        """

        print("{:<25} {:<30} {:<20} {:<20} {:<20}".format('Book Id','Book Title','Book Author','Book Price','Books Available'))
        print("{:<25} {:<30} {:<20} {:<20} {:<20}".format('--------','-----------','-----------','-----------','--------------'))

        try:

            with open(self.file_name,'r') as fp:
                reader = csv.reader(fp)
                for row in reader:
                    print("{:<25} {:<30} {:<20} {:<20} {:<20}".format(row[0],row[1],row[2],round(float(row[3]),2),row[4]))

        except FileNotFoundError:

            print("No Books Data is Available, Please Add Books Data")
            print()

        except IndexError:
            pass


    def search_book(self):

        """
            search for a book in file with specified book id provided in input and print the book details,
            else print no data found
        """

        search_text = input('Enter Book Name / Author Name: ')

        with open(self.file_name,'r') as fp:
            reader = csv.reader(fp)

            print("{:<25} {:<30} {:<20} {:<20} {:<20}".format('Book Id','Book Title','Book Author','Book Price','Books Available'))
            print("{:<25} {:<30} {:<20} {:<20} {:<20}".format('--------','-----------','-----------','-----------','--------------'))

            for row in reader:
                for col in row:
                    if search_text in col:
    
                        print("{:<25} {:<30} {:<20} {:<20} {:<20}".format(row[0],row[1],row[2],round(float(row[3]),2),row[4]))
                        print()
                        break

            else:
                # logging.warning('Book not found')
                print()



    def remove_book(self):
        
        """
            remove a book from file with specified book id provided in input.
        """

        try:

            book_id = input('Enter Book Id: ')
            lines = []

            with open(self.file_name,'r') as fp:
                reader = csv.reader(fp)
                for row in reader:
                    lines.append(row)
                    if row[0] == book_id:
                        lines.remove(row)
                
                else:

                    print(f"Book Not Found with Book id {book_id}")
                    print()
                    
            with open(self.file_name,'w',newline='') as fp:
                writer = csv.writer(fp)
                for line in lines:
                    writer.writerow(line)
            try:

                self.list_books()

            except IndexError:
                pass

        except FileNotFoundError:

            print("No Books Data is Available, Please Add Books Data")
            print()



    def show_inventory(self):

        """"
            Method for displaying Inventory Data .
        """
        books_sum = 0
        total_price = 0

        title = " Bookstore Inventory Management System "
        title = title.upper()
        print(title.center(150,'-'))

        now = datetime.now()
        dt = now.strftime("%d/%m/%Y %H:%M:%S")

        print(f'Date: {dt.center(260," ")}')
        print()

        print("{:<20} {:<30} {:<25} {:<20} {:<20} {:<30}".format('Book Id','Book Title','Book Price(Per Unit)','Books Available','Total','Book Added Date-Time'))
        print("{:<20} {:<30} {:<25} {:<20} {:<20} {:<30}".format('--------','-----------','-----------','-----------','-----------','-----------------------'))

        try:

            with open(self.file_name,'r') as fp:
                reader = csv.reader(fp)
                for row in reader:
                    print("{:<20} {:<30} {:<25} {:<20} {:<20} {:<30}".format(row[0],row[1],round(float(row[3]),2),row[4],round(float(row[3])*float(row[4]),2),row[5]))
                    books_sum+= int(row[4])
                    total_price+= float(row[3])*float(row[4])

            print()
            print("{:<20} {:<30} {:<20} {:<25} {:<20} ".format('             ','TOTAL BOOKS',books_sum,'TOTAL PRICE',round(total_price,2)))    
            print()

        except FileNotFoundError:

            print("No Books Data is Available, Please Add Books Data")
            print()



    def main(self):

        """
            displays the available options to the user to Perform Operations.
        """


        while True:
            print()
            print("Welcome to Book Inventary Management System(BIMS)")
            print()
            user_option = input('1.Add Book  \n2.Show Books \n3.Search Book \n4.Delete Book \n5.Show Inventary \n6.Quite \nEnter your Option: ')
            print()

            if user_option == '1':
                self.add_book()
            
            elif user_option == '2':
                self.list_books()
            
            elif user_option == '3':
                self.search_book()
                
            elif user_option == '4':
                self.remove_book()

            elif user_option == '5':
                self.show_inventory()

            elif user_option == '6':
                break
            
            else:
                print('Invalid Option')


# Creating Object of Inventary Class
if __name__ == '__main__':
    inventary_obj = Inventary()
    inventary_obj.main()


# {:<20} means left-aligned with width 20



