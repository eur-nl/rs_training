
# Automating Tasks with Python

In this second session of the Programming Café, we’ll dive into how to use Python scripts to automate repetitive tasks in your research. We’ll explore the basics of automation, focusing on how to effectively move, rename and copy your files and write clear, functional scripts.

We will start with a question: Which tasks would you like to automate in your research ?
I hope that I can show you some tricks for these tasks today.

## Requirements to start

Did everyone install Python ? Who needs help ?
I would recommend the use of PyCharm!
- Please create a copy of the files that you want to work on.
- You can also download test files extracted from the [German Summary Corpus](https://clarin.eurac.edu/repository/xmlui/handle/20.500.12124/81) (Wedig & Strobl, 2024) **[here](extras/data.zip)**. 
- Create a folder "data" in your projects and a subfolder "files" in which you paste your files

## Import the necessary modules and create a main function to coordinate all tasks
```python
import os  #package that gives us access to the operating system
import shutil #package that gives us access to file operations
import re #package that gives us access to regular expressions
import time #package that gives us access to time

if __name__ == '__main__':
  # will be filled with our functions
    
```
## File Management

During a busy period like a PhD, tasks such as tracking existing files or creating backups may be overlooked. Python can help automate these tasks, allowing you to complete them with the press of a button.

### Generate a file structure automatically

At the beginning of your project (or at a later stage), you might wonder how to best structure your files. This code snippet allows you to automate the process. **Be aware: data/copy will be created for this session today and is not typical in a research project**
```python
def create_folders():
  folders = ["data/raw", "data/processed", "data/copy" "scripts", "results"] # Here you can define how you would like to name your folders
  # Create folders
  for folder in folders:
      os.makedirs(folder, exist_ok=True)
```
### Get an overview of all files in one folder
To get an overview of all the files in a folder, you can use the listdir() function.

```python
def list_files():
  # Get the directory path
  working_dir = os.getcwd()
  path = os.path.join(working_dir, "data", "files")
  
  print(os.listdir(path)) # print allows us to see the results on the console.
```

### Copying files: e.g., to create a backup

You might find yourself in a situation where you want to create copies of your files, such as for backup purposes. For this, you can use the shutil package.
```python
def copy_files():
  # Get the directory path
  working_dir = os.getcwd() 
  path = os.path.join(working_dir, "data", "files") 
  
  # this is how we copy one file
  source = os.path.join(path, "L1_Ki_02.csv") 
  destination = os.path.join(working_dir, "data", "test", "copy_L1_Ki_02.csv")
  shutil.copyfile(source, destination)
  
  # we can also copy a while directory
  shutil.copytree(os.path.join(working_dir, "data", "files"), os.path.join(working_dir, "data", "copy"), dirs_exist_ok = True)
```

### Renaming files
After copying the files, we can perform some small operations on them!

Have you ever found yourself in a situation where you realized you didn't follow a clear concept in your file naming or ordered the sequences within the file names incorrectly? Let's fix that!

Let's start simple: add "exp" in front of the current file name.
```python
def change_name():
  # Get the directory path
  working_dir = os.getcwd() 
  folder_path = os.path.join(working_dir, "data","copy") 
  
  #for every file in that folder, we want to create a new file name and then use rename to change the name
  for file in os.listdir(folder_path): 
    new_name = "exp_" + file
    os.rename(os.path.join(folder_path, file), os.path.join(folder_path, new_name))
```

Now a bit more complex, let's only change csv files
```python
def change_name_csv():
  # Get the directory path
  working_dir = os.getcwd()
  folder_path = os.path.join(working_dir, "data","copy") 
  
  # for every file, we want to change, but only if the file ends with ".csv"
  for file in os.listdir(folder_path):
      if file.endswith(".csv"):
        new_name = "csv_" + file
        os.rename(os.path.join(folder_path,file), os.path.join(folder_path,new_name))
```

We can also make it a bit more advanced here and only add the string "csv" where "csv" is not added (or included in the name already)
```python
def change_name_csv_advanced():
  # Get the directory path
  working_dir = os.getcwd()
  folder_path = os.path.join(working_dir, "data","copy") 
  
  # for every file, we want to change, but only if the file ends with ".csv" and does not include csv already
  for file in os.listdir(folder_path):
      if file.endswith(".csv") and "csv" not in file:
        new_name = "csv_" + file
        os.rename(os.path.join(folder_path, file), os.path.join(folder_path, new_name))
```

We can also adjust this in more detail using regular expressions

```python
def change_name_re():
  # Get the directory path
  working_dir = os.getcwd()
  folder_path = os.path.join(working_dir, "data","copy")
  
  # Create a pattern (\s means whitespace)
  pattern = re.compile(r"(\s+)")
  # and define what you want to replace it with
  replace_with = "_"
  
  # for every file, check whether it fits the pattern and change the name accordingly
  for file in os.listdir(folder_path):
      new_name = pattern.sub(replace_with, file)
      os.rename(os.path.join(folder_path, file), os.path.join(folder_path, new_name))
```

Regular expressions are not limited to spaces, you can adapt them to match your file names. Best to use is **[regexr](https://regexr.com)** for this. They also provide a handy cheatsheet.

### Moving files based on criteria - Sort your files
Another task that you may be interested in is sorting files. To sort files, we can use shutil again. We can sort files according to various criteria. For now, I show you file types and file names.

#### Sort all your files based on file type
To sort files by file type, we can use the .endswith() function that we already used previously. 
```python
def sort_files_type():
  # Get the directory path
  working_dir = os.getcwd()
  folder_path = os.path.join(working_dir, "data","copy")
  
  # Decide on the names of the new directories (can also be nicer, but quick and dirty now)
  destination_csv = os.path.join(working_dir, "data","csv_files")
  destination_txt = os.path.join(working_dir, "data","txt_files")
  
  # Create the directories
  os.makedirs(destination_csv, exist_ok=True)
  os.makedirs(destination_txt, exist_ok=True)
  
  # for every file in the directory, sort in the right new folder
  for file in os.listdir(folder_path):
      if file.endswith(".csv"):
          shutil.move(os.path.join(folder_path, file), os.path.join(destination_csv, file))
      elif file.endswith(".txt"):
          shutil.move(os.path.join(folder_path, file), os.path.join(destination_txt, file))
```

#### Sort your files based on file name
To sort the files by name or certain characteristics of the names, we use the "in" operator.

```python
def sort_file_name():
  # Get the directory path
  working_dir = os.getcwd()
  folder_path = os.path.join(working_dir, "data","copy")
  
  # Decide on the names of the new directories (can also be nicer, but quick and dirty now)
  destination_L1 = os.path.join(working_dir, "data","L1")
  destination_L2 = os.path.join(working_dir, "data","L2")
  os.makedirs(destination_L1, exist_ok=True)
  os.makedirs(destination_L2, exist_ok=True)
  
  # for every file in the directory, sort in the right new folder using "in"
  for file in os.listdir(folder_path):
      if "L1" in file:
          shutil.move(os.path.join(folder_path, file), os.path.join(destination_L1, file))
      elif "L2" in file:
        shutil.move(os.path.join(folder_path, file), os.path.join(destination_L2, file))
```

## Documentation
For a research project, you might also want a list of all files along with their modification dates. We can use Python for that!

### Generate a README with all file names and their last modification date
```python
def generate_README():
  # Get the directory path
  working_dir = os.getcwd()
  folder_path = os.path.join(working_dir, "data")
  
  # Open the Readme file, print the header and then all information about the files
  with open("README.txt", mode = "w", encoding = "utf-8") as file_out:
      print("directory", "subdirectory", "file", "modification_date", sep = "\t", file = file_out)
      # for every directory/folder that we find...
      for dirpath, dirnames, files in os.walk(folder_path):
      #...iterate through files and print information in the readme
          for file in files:
              path =  os.path.join(dirpath, file)
              mod_time = time.ctime(os.path.getmtime(path))
              print(os.path.basename(folder_path), os.path.basename(dirpath), file, mod_time, sep = "\t", file = file_out)
```

In the end, our main function may look like this:

```python
if __name__ == '__main__':
  create_folders()
  list_files()
  copy_files()
  change_name()
  change_name_csv()
  change_name_csv_advanced()
  sort_file_name()
  sort_file_type()
  generate_README()
```

I hope that this already helps you a bit for your research projects!


Any other task that you want to optimize/automate?

**[Here](https://automatetheboringstuff.com)** is a nice resource with even more topics!
