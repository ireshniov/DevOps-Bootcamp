******

<details>
<summary>Exercise 1: Working with Lists </summary>
 <br />

```python
my_list = [1, 2, 2, 4, 4, 5, 6, 8, 10, 13, 22, 35, 52, 83]

# Write a program that prints out all the elements of the list that are higher than or equal to 10.
print([x for x in my_list if x >= 10])

# Instead of printing the elements one by one, make a new list that has all the elements higher than or equal to 10 from this list in it and print out this new list.
new_list = [x for x in my_list if x >= 10]
print(new_list)

# Ask the user for a number as input and print a list that contains only those elements from my_list that are higher than the number given by the user.
number = input("Enter a number: ")
print([x for x in my_list if x >= int(number)])
```

</details>

******

<details>
<summary>Exercise 2: Working with Dictionaries </summary>
 <br />

```python
employee = {
  "name": "Tim",
  "age": 30,
  "birthday": "1990-03-10",
  "job": "DevOps Engineer"
}

# Write a Python Script that:
# Updates the job to Software Engineer
employee.update({ "job": "Software Engineer" })

# Removes the age key from the dictionary
employee.pop("age")

# Loops through the dictionary and prints the key:value pairs one by one
for item in employee.items():
    print(f"{item[0]}:{item[1]}")

dict_one = {'a': 100, 'b': 400}
dict_two = {'x': 300, 'y': 200}

# Write a Python Script that:
# Merges these two Python dictionaries into 1 new dictionary.
merged_dict = {**dict_one, **dict_two}
# merged_dict = dict_one | dict_two

# Sums up all the values in the new dictionary and prints it out
print(f"Sum of values: {sum(merged_dict.values())}")

# Prints the max and minimum values of the dictionary
print(f"Minimum value: {min(merged_dict.values())}")
print(f"Maximum value: {max(merged_dict.values())}")
```

</details>

******

<details>
<summary>Exercise 3: Working with List of Dictionaries </summary>
 <br />

```python
employees = [{
  "name": "Tina",
  "age": 30,
  "birthday": "1990-03-10",
  "job": "DevOps Engineer",
  "address": {
    "city": "New York",
    "country": "USA"
  }
},
{
  "name": "Tim",
  "age": 35,
  "birthday": "1985-02-21",
  "job": "Developer",
  "address": {
    "city": "Sydney",
    "country": "Australia"
  }
}]

# Write a Python Program that:
# Prints out - the name, job and city of each employee using a loop. The program must work for any number of employees in the list, not just 2.
output_list = ([f"{employee.get('name')}, {employee.get('job')}, {employee.get('address').get('city')}" for employee in employees ])
for output in output_list:
    print(f"{output}")

# Prints the country of the second employee in the list by accessing it directly without the loop.
print(employees[1].get('address').get('country'))
```

</details>

******

<details>
<summary>Exercise 4: Working with Functions </summary>
 <br />

`task4_module.py`
```
# For cleaner code, declare these functions in its own helper Module and use them in the main.py file
from typing import Dict, List, NamedTuple

# Write a function that accepts a list of dictionaries with employee age (see example list from Exercise 3) and prints out the name and age of the youngest employee.
def get_youngest_employer(employees: List[Dict[str, int]]) -> Dict[str, int]:
    return min(employees, key=lambda x: x['age'])

# Write a function that accepts a string and calculates the number of upper case letters and lower case letters.
class UpperLower(NamedTuple):
    upper: int
    lower: int

def number_of_upper_lower(input: str) -> UpperLower:
    filtered_input = input.strip().replace(" ", "")
    upper = sum(1 for letter in filtered_input if letter.isupper())
    lower = sum(1 for letter in filtered_input if letter.islower())
    return UpperLower(upper, lower)

# Write a function that prints the even numbers from a provided list.
def get_even_numbers(numbers: List[int]) -> List[int]:
    return [number for number in numbers if number % 2 == 0]
```

```python
from task4_module import get_youngest_employer, number_of_upper_lower, get_even_numbers

employees = [{
  "name": "Tina",
  "age": 30
},
{
  "name": "Tim",
  "age": 35
}]


print(get_youngest_employer(employees))

print(number_of_upper_lower("THIs iS test"))

print(get_even_numbers([1, 4, 3, 7, 5, 6, 8]))
```

</details>

******

<details>
<summary>Exercise 5: Python Program 'Calculator' </summary>
 <br />

```python
# Write a simple calculator program that:
#
# takes user input of 2 numbers and operation to execute
# handles the following operations: plus, minus, multiply, divide
# does proper user validation and give feedback: only numbers allowed
# Keeps the Calculator program running until the user types “exit”
# Keeps track of how many calculations the user has taken, and when the user exits the calculator program, prints out the number of calculations the user did

class Calculator:
    calculations: int

    def __init__(self):
        self.calculations = 0

    def add(self, a: float, b: float) -> float:
        if not isinstance(a, float) or not isinstance(b, float):
            raise TypeError('a and b must be floats')

        self.calculations += 1
        return a + b

    def subtract(self, a: float, b: float) -> float:
        if not isinstance(a, float) or not isinstance(b, float):
            raise TypeError('a and b must be floats')

        self.calculations += 1
        return a - b

    def multiply(self, a: float, b: float) -> float:
        if not isinstance(a, float) or not isinstance(b, float):
            raise TypeError('a and b must be floats')

        self.calculations += 1
        return a * b

    def divide(self, a: float, b: float) -> float:
        if not isinstance(a, float) or not isinstance(b, float):
            raise TypeError('a and b must be floats')

        if b == 0:
            raise ZeroDivisionError('division by zero')

        self.calculations += 1
        return a / b

    def get_calculations(self) -> int:
        return self.calculations


calculator = Calculator()

while True:
    number1 = input('Enter first number: ')

    if (number1 == 'exit') or (number1 is None):
        print('Bye!')
        print(f"Number of calculations: {calculator.get_calculations()}")
        break

    number2 = input('Enter second number: ')

    try:
        number1 = float(number1)
        number2 = float(number2)
    except ValueError:
        raise TypeError('number1 and number2 must be floats')

    # if not isinstance(number1, float) or not isinstance(number2, float):
    #     raise TypeError('number1 and number2 must be floats')

    operation = input('Enter operation: ')

    result: float = 0
    if operation == "add":
        result = calculator.add(number1, number2)
    elif operation == "subtract":
        result = calculator.subtract(number1, number2)
    elif operation == "multiply":
        result = calculator.multiply(number1, number2)
    elif operation == "divide":
        result = calculator.divide(number1, number2)
    else:
        print("Not a valid operation")

    print(f"Result: {result}")
```

</details>

******

<details>
<summary>Exercise 6: Python Program 'Guessing Game' </summary>
 <br />

```python
# Write a program that:
#
# runs until the user guesses a number (hint: while loop)
# generates a random number between 1 and 9 (including 1 and 9)
# asks the user to guess the number
# then prints a message to the user, whether they guessed too low, too high
# if the user guesses the number right, print out YOU WON! and exit the program
from random import randrange

while True:
    random = randrange(0, 10)

    try:
        answer = int(input("Guess a number: "))
    except ValueError:
        print("Please enter a number")
        continue

    if answer == random:
        print("YOU WON!")
        break

    if answer > random:
        print("Number is too high")
        continue

    if answer < random:
        print("Number is too low")
        continue
```

</details>

******

<details>
<summary>Exercise 7: Working with Classes and Objects </summary>
 <br />

```python
from typing import List

class Person:
    first_name: str
    last_name: str
    age: int

    def __init__(self, first_name: str, last_name: str, age: int) -> None:
        self.first_name = first_name
        self.last_name = last_name
        self.age = age

    def get_full_name(self):
        return f"{self.first_name} {self.last_name}"

class Lecture:
    name: str
    max_students_number: int
    duration: float

    def __init__(self, name: str, max_students_number: int, duration: float) -> None:
        self.name = name
        self.max_students_number = max_students_number
        self.duration = duration

    def get_name(self) -> str:
        return self.name

    def get_duration(self) -> float:
        return self.duration

class Professor(Person):
    lectures: List[Lecture] = []

    def __init__(self, first_name: str, last_name: str, age: int) -> None:
        super().__init__(first_name, last_name, age)

    def add_lecture(self, lecture: Lecture) -> None:
        self.lectures.append(lecture)

    def remove_lecture(self, lecture: Lecture) -> None:
        self.lectures.remove(lecture)

    def get_lectures(self)-> List[Lecture]:
        return self.lectures

class Student(Person):
    lectures: List[Lecture] = []

    def __init__(self, first_name: str, last_name: str, age: int) -> None:
        super().__init__(first_name, last_name, age)

    def add_lecture(self, lecture: Lecture):
        self.lectures.append(lecture)

    def remove_lecture(self, lecture: Lecture):
        self.lectures.remove(lecture)

    def get_lectures(self)-> List[Lecture]:
        return self.lectures

cs_lecture = Lecture("Computer science", 15, 45)
python_basics_lecture = Lecture("Python programming basics", 25, 90)
python_advanced_lecture = Lecture("Python advanced", 10, 90)
algorithms_lecture = Lecture("Algorithms and data sturctures", 30, 120)

new_professor = Professor("Maria", "Smith", 34)
new_professor.add_lecture(cs_lecture)
new_professor.add_lecture(python_basics_lecture)

print(new_professor.get_full_name())
new_professor.add_lecture(python_advanced_lecture)

for lecture in new_professor.get_lectures():
    print(f"- {lecture.name}")

print("------------------------------")

new_student = Student("David", "Green", 25)
new_student.add_lecture(algorithms_lecture)
print(new_student.get_full_name())
new_student.add_lecture(python_basics_lecture)
for lecture in new_student.get_lectures():
    print(f"- {lecture.name}")

print("------------------------------")

print(f"{cs_lecture.name} - {cs_lecture.duration} minutes")
print(f"{python_basics_lecture.name} - {python_basics_lecture.duration} minutes")
```

</details>

******

<details>
<summary>Exercise 8: Working with Dates </summary>
 <br />

```python
# Write a program that:
# accepts user's birthday as input
# and calculates how many days, hours and minutes are remaining till the birthday
# prints out the result as a message to the user

from datetime import datetime

birthday = input('Enter your birthday (like 09-03-1993: ')

birthday_date = datetime.strptime(birthday, "%d-%m-%Y").date()
timenow = datetime.today()

year = timenow.year

while True:
    next_birthday = datetime(year, birthday_date.month, birthday_date.day)
    if next_birthday.date() > timenow.date():
        print(f"{ (next_birthday-timenow).days } days till your birthday")
        break
    year += 1
```

</details>

******

<details>
<summary>Exercise 9: Working with Spreadsheets </summary>
 <br />

```python
# Write a program that:
# reads the provided spreadsheet file "employees.xlsx" (see Download section at the bottom) with the following information/columns: "name", "years of experience", "job title", "date of birth"
# creates a new spreadsheet file "employees_sorted.xlsx" with following info/columns: "name", "years of experience", where the years of experience is sorted in descending order: so the employee name with the most experience in years is on top.

from typing import NamedTuple, List

import openpyxl
from openpyxl.cell import Cell
from openpyxl.workbook import Workbook
from openpyxl.worksheet.worksheet import Worksheet


class Employee(NamedTuple):
    name: str
    exp: int

wb: Workbook = openpyxl.load_workbook('employees.xlsx')
ws: Worksheet = wb.active

new_dict: List[Employee] = []

for row in range(2, ws.max_row + 1):
    name: Cell = ws.cell(row=row, column=1).value
    exp: Cell = ws.cell(row=row, column=2).value

    new_dict.append(Employee(name, int(exp)))

sorted_dict: List[Employee] = sorted(new_dict, key=lambda x: x.exp, reverse=True)

print(sorted_dict)

for row in range(2, ws.max_row + 1):
    name: Cell = ws.cell(row=row, column=1)
    exp: Cell = ws.cell(row=row, column=2)

    employee: Employee = sorted_dict[row - 2]

    name.value = employee.name
    exp.value = employee.exp

wb.save('employees_sorted_by_experience.xlsx')
```

</details>

******

<details>
<summary>Exercise 10: Working with REST APIs </summary>
 <br />

```python
# Write a program that:
# connects to GitHub API
# gets all the public repositories for a specific GitHub user
# prints the name & URL of every project

import requests

user = "ireshniov"
data = requests.get(f"https://api.github.com/users/{user}/repos").json()

for project in data:
    print(f"Name: {project['name']}\nUrl: {project['html_url']}\n")
```

</details>

******
