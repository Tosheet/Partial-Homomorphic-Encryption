import os
from cryptography.fernet import Fernet
from functools import reduce
import tkinter as tk
from tkinter import filedialog
from tkinter import messagebox
import datetime

# Create a window object
window = tk.Tk()

# Set the window title
window.title("MedLock")

# Set the window size
window.geometry("500x300")

# Load the image file
image_file = tk.PhotoImage(file="D:\DYPIU\Fourth Year\8th Semester\Code\Logo1.png")

# Create a label with the image as the background
background_label = tk.Label(window, image=image_file)
background_label.place(relx=0.5, rely=0.5, anchor=tk.CENTER)

# Add other widgets to the window
label = tk.Label( text="WELCOME")
label.pack()

# Define global variables
key = None
fernet = None
dir_path = None


# Function to generate a key and create a Fernet object
def generate_key():
    global key, fernet
    key = Fernet.generate_key()
    fernet = Fernet(key)

    # Add timestamp to the key
    timestamp = datetime.datetime.now().strftime("%d-%m-%Y %H:%M:%S")
    key_with_timestamp = f"{key.decode('utf-8')} - {timestamp}"

    with open('key.txt', 'a') as file:
        file.write(key_with_timestamp + '\n')
    messagebox.showinfo("Info", "Key generated successfully.")


# Function to select a directory
def select_directory():
    global dir_path
    dir_path = filedialog.askdirectory()
    dir_path_label.config(text=dir_path)

# Function to encrypt a file
def encrypt_file(file_path):
    try:
        with open(file_path, 'rb') as file:
            file_data = file.read()
            encrypted_data = fernet.encrypt(file_data)

            # Get the base file name and extension
            file_name, file_extension = os.path.splitext(file_path)

            # Save the encrypted data to a file
            with open(f"{file_name}.encrypted", 'wb') as encrypted_file:
                encrypted_file.write(encrypted_data)

            message = f"{file_path} encrypted successfully."
    except:
        message = f"Error: Failed to encrypt {file_path}."
    return message

# Function to decrypt a file using all keys in a txt file
def decrypt_file(file_path, output_dir):
    try:
        with open(file_path, 'rb') as encrypted_file:
            encrypted_data = encrypted_file.read()

            with open('key.txt', 'r') as key_file:
                for key in key_file:
                    try:
                        fernet = Fernet(key.strip().encode('utf-8'))
                        decrypted_data = fernet.decrypt(encrypted_data)

                        # Save the decrypted data to a file
                        output_file_path = os.path.join(output_dir, os.path.basename(file_path)[:-10] + '.decrypted')
                        with open(output_file_path, 'wb') as decrypted_file:
                            decrypted_file.write(decrypted_data)

                        message = f"{file_path} decrypted successfully."
                        return message
                    except:
                        pass
            message = f"Error: Failed to decrypt {file_path}."
    except:
        message = f"Error: Failed to decrypt {file_path}."
    return message


# Function to encrypt all files in the directory
def encrypt_directory():
    global dir_path
    if dir_path is None:
        messagebox.showerror("Error", "Please select a directory.")
    else:
        file_paths = [os.path.join(dir_path, f) for f in os.listdir(dir_path) if os.path.isfile(os.path.join(dir_path, f))]
        messages = []
        file_types = {}
        for file_path in file_paths:
            if not file_path.endswith('.encrypted'):
                message = encrypt_file(file_path)
                messages.append(message)
                file_extension = os.path.splitext(file_path)[1].lower()
                if file_extension not in file_types:
                    file_types[file_extension] = 1
                else:
                    file_types[file_extension] += 1
        if messages:
            messagebox.showinfo("Info", "\n".join(messages))
            messagebox.showinfo("Info", f"Encrypted {len(messages)} files. File types: {file_types}")
        else:
            messagebox.showinfo("Info", "No files to encrypt.")

# Function to decrypt all files in the directory
def decrypt_directory():
    global dir_path
    if dir_path is None:
        messagebox.showerror("Error", "Please select a directory.")
    else:
        output_dir = filedialog.askdirectory()
        if output_dir:
            file_paths = [os.path.join(dir_path, f) for f in os.listdir(dir_path) if os.path.isfile(os.path.join(dir_path, f))]
            messages = []
            file_types = {}
            for file_path in file_paths:
                if file_path.endswith('.encrypted'):
                    message = decrypt_file(file_path, output_dir)
                    messages.append(message)
                    file_extension = os.path.splitext(file_path)[1].lower()
                    if file_extension not in file_types:
                        file_types[file_extension] = 1
                    else:
                        file_types[file_extension] += 1
            if messages:
                messagebox.showinfo("Info", "\n".join(messages))
                messagebox.showinfo("Info", f"Decrypted {len(messages)} files. File types: {file_types}")
            else:
                messagebox.showinfo("Info", "No files to decrypt.")

def count_files():
    global dir_path
    if dir_path is None:
        messagebox.showerror("Error", "Please select a directory.")
    else:
        file_counts = {}
        for root, dirs, files in os.walk(dir_path):
            for file in files:
                file_ext = os.path.splitext(file)[1]
                if file_ext not in file_counts:
                    file_counts[file_ext] = 1
                else:
                    file_counts[file_ext] += 1


# Create a label for the directory path
dir_path_label = tk.Label(window, text="No directory selected.", font=("Arial", 12))
dir_path_label.pack(pady=20)

# Create a button to select a directory
select_dir_button = tk.Button(window, text="Select Directory", command=select_directory)
select_dir_button.pack()

# Create a button to generate a key
generate_key_button = tk.Button(window, text="Generate Key", command=generate_key)
generate_key_button.pack(pady=20)

# Create a button to encrypt all files in the directory
encrypt_button = tk.Button(window, text="Encrypt", command=encrypt_directory)
encrypt_button.pack(pady=20)

# Create a button to decrypt all files in the directory
decrypt_button = tk.Button(window, text="Decrypt", command=decrypt_directory)
decrypt_button.pack(pady=20)

# Run the window
window.mainloop()

