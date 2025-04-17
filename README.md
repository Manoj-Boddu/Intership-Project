# Intership-Project
#Currency Converter with GUI
import tkinter as tk
import requests

def convert_currency():
    try:
        amount = float(amount_entry.get())
        from_currency = from_currency_entry.get().upper()
        to_currency = to_currency_entry.get().upper()

        # Replace with a real API endpoint
        api_url = f"https://api.exchangerate-api.com/v4/latest/{from_currency}"  # A more common API
        response = requests.get(api_url)
        data = response.json()
        exchange_rate = data['rates'][to_currency]
        converted_amount = amount * exchange_rate
        result_label.config(text=f"Converted Amount: {converted_amount:.2f} {to_currency}")
    except ValueError:
        result_label.config(text="Invalid input. Please enter a number for amount.")
    except requests.exceptions.RequestException as e:
        result_label.config(text=f"API Error: {e}")
    except KeyError:
        result_label.config(text=f"Currency code {to_currency} not found.")
    except Exception as e:
        result_label.config(text=f"Error: {e}")

# --- GUI Setup ---
window = tk.Tk()
window.title("Currency Converter")

# --- Widgets ---
amount_label = tk.Label(window, text="Amount:")
amount_entry = tk.Entry(window)
from_currency_label = tk.Label(window, text="From Currency (e.g., USD):")
from_currency_entry = tk.Entry(window)
to_currency_label = tk.Label(window, text="To Currency (e.g., EUR):")
to_currency_entry = tk.Entry(window)

convert_button = tk.Button(window, text="Convert", command=convert_currency)
result_label = tk.Label(window, text="Converted Amount:")

# --- Layout using grid() ---
amount_label.grid(row=0, column=0, padx=5, pady=5)
amount_entry.grid(row=0, column=1, padx=5, pady=5)
from_currency_label.grid(row=1, column=0, padx=5, pady=5)
from_currency_entry.grid(row=1, column=1, padx=5, pady=5)
to_currency_label.grid(row=2, column=0, padx=5, pady=5)
to_currency_entry.grid(row=2, column=1, padx=5, pady=5)
convert_button.grid(row=3, column=0, columnspan=2, padx=5, pady=5)
result_label.grid(row=4, column=0, columnspan=2, padx=5, pady=5)

window.mainloop()  # Start the GUI event loop
import tkinter as tk
import random
import string

def generate_password():
    try:
        length = int(length_entry.get())
        if length < 8:  # Minimum reasonable password length
            result_label.config(text="Password length should be at least 8.")
            return

        characters = string.ascii_letters + string.digits + string.punctuation
        password = ''.join(random.choice(characters) for i in range(length))
        password_entry.delete(0, tk.END)
        password_entry.insert(0, password)
        result_label.config(text="Password generated!")
    except ValueError:
        result_label.config(text="Invalid input. Please enter a number for length.")

# --- GUI Setup ---
window = tk.Tk()
window.title("Password Generator")

# --- Widgets ---
length_label = tk.Label(window, text="Password Length:")
length_entry = tk.Entry(window)
generate_button = tk.Button(window, text="Generate Password", command=generate_password)
password_entry = tk.Entry(window, show="*")  # Show asterisks
result_label = tk.Label(window, text="")

# --- Layout ---
length_label.grid(row=0, column=0, padx=5, pady=5)
length_entry.grid(row=0, column=1, padx=5, pady=5)
generate_button.grid(row=1, column=0, columnspan=2, padx=5, pady=5)
password_entry.grid(row=2, column=0, columnspan=2, padx=5, pady=5)
result_label.grid(row=3, column=0, columnspan=2, padx=5, pady=5)

window.mainloop()
import speech_recognition as sr
import pyttsx3
import sqlite3
import threading  # For handling voice input in a separate thread
import tkinter as tk  # For a basic GUI to start/stop recording

class AttendanceRecorder:
    def __init__(self, window):
        self.window = window
        self.recognizer = sr.Recognizer()
        self.engine = pyttsx3.init()
        self.conn = sqlite3.connect('attendance.db')
        self.cursor = self.conn.cursor()
        self.cursor.execute('''CREATE TABLE IF NOT EXISTS attendance (name TEXT, timestamp DATETIME DEFAULT CURRENT_TIMESTAMP)''')
        self.is_recording = False
        self.setup_gui()

    def setup_gui(self):
        self.start_button = tk.Button(self.window, text="Start Recording", command=self.start_recording)
        self.stop_button = tk.Button(self.window, text="Stop Recording", command=self.stop_recording, state=tk.DISABLED)  # Initially disabled
        self.log_text = tk.Text(self.window, height=10, width=40)
        self.start_button.pack(pady=10)
        self.stop_button.pack(pady=10)
        self.log_text.pack(pady=10)

    def start_recording(self):
        self.is_recording = True
        self.start_button.config(state=tk.DISABLED)
        self.stop_button.config(state=tk.NORMAL)
        threading.Thread(target=self.record_loop, daemon=True).start()  # Start recording in a thread

    def stop_recording(self):
        self.is_recording = False
        self.start_button.config(state=tk.NORMAL)
        self.stop_button.config(state=tk.DISABLED)

    def record_loop(self):
        while self.is_recording:
            with sr.Microphone() as source:
                self.log_message("Listening for name...")
                try:
                    audio = self.recognizer.listen(source, timeout=5)  # Adjust timeout as needed
                    name = self.recognizer.recognize_google(audio)
                    self.record_attendance(name)
                except sr.WaitTimeoutError:
                    pass  # No speech detected within timeout, continue loop
                except sr.UnknownValueError:
                    self.log_message("Could not understand audio")
                except sr.RequestError as e:
                    self.log_message(f"Speech Recognition error: {e}")
                    self.stop_recording()  # Stop if there's an API error
                    break
                except Exception as e:
                    self.log_message(f"An unexpected error occurred: {e}")
                    self.stop_recording()
                    break

    def record_attendance(self, name):
        self.cursor.execute("INSERT INTO attendance (name) VALUES (?)", (name,))
        self.conn.commit()
        self.log_message(f"Attendance recorded for: {name}")
        self.engine.say(f"Attendance recorded for {name}")
        self.engine.runAndWait()

    def log_message(self, message):
        self.log_text.insert(tk.END, message + "\n")
        self.log_text.see(tk.END)  # Scroll to the bottom

    def close_connection(self):
        self.conn.close()

if __name__ == "__main__":
    window = tk.Tk()
    window.title("Voice Attendance")
    recorder = AttendanceRecorder(window)
    window.protocol("WM_DELETE_WINDOW", recorder.close_connection)  # Close DB on window close
    window.mainloop()
