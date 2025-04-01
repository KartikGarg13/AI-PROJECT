# AI-PROJECT
AI LANGUAGE TRANSLATOR 



import os
import tkinter as tk
from tkinter import ttk, messagebox
from PIL import Image, ImageTk
import requests
import json

# 🔹 Replace with your actual API key
DEEPSEEK_API_KEY = "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"

# 🔹 Supported languages
LANGUAGES = ["Afrikaans", "Albanian", "Amharic", "Arabic", "Armenian", "Azerbaijani", "Basque", "Bengali",
             "Bulgarian", "Catalan", "Chinese (Simplified)", "Chinese (Traditional)", "Croatian", "Czech",
             "Danish", "Dutch", "English", "Estonian", "Filipino", "Finnish", "French", "Galician", "Georgian",
             "German", "Greek", "Gujarati", "Hebrew", "Hindi", "Hungarian", "Icelandic", "Indonesian",
             "Italian", "Japanese", "Javanese", "Kannada", "Korean", "Latvian", "Lithuanian", "Malay",
             "Malayalam", "Marathi", "Mongolian", "Nepali", "Norwegian", "Pashto", "Persian", "Polish",
             "Portuguese", "Punjabi", "Romanian", "Russian", "Serbian", "Slovak", "Slovenian", "Spanish",
             "Swahili", "Swedish", "Tamil", "Telugu", "Thai", "Turkish", "Ukrainian", "Urdu", "Vietnamese"]

# 🔹 Store Chat History
chat_history = []

# 🔹 Function to Enhance Text using AI
def enhance_text():
    text = text_input.get("1.0", tk.END).strip()
    if not text:
        messagebox.showerror("Error", "Please enter text to enhance.")
        return

    try:
        response = requests.post(
            url="https://openrouter.ai/api/v1/chat/completions",
            headers={"Authorization": f"Bearer {DEEPSEEK_API_KEY}", "Content-Type": "application/json"},
            data=json.dumps({
                "model": "deepseek/deepseek-r1:free",
                "messages": [
                    {"role": "system", "content": "You are an AI that improves text clarity and fixes grammar."},
                    {"role": "user", "content": f"Fix grammar and spelling: {text}"}
                ]
            })
        )

        result = response.json()
        if "choices" in result:
            improved_text = result["choices"][0]["message"]["content"].strip()
            text_output.delete("1.0", tk.END)
            text_output.insert(tk.END, improved_text)
            add_to_history(f"Enhanced: {text[:30]}...", improved_text)  # Store history
        else:
            raise Exception(result.get("error", "Unknown error"))
    except Exception as e:
        messagebox.showerror("AI Enhancement Error", str(e))

# 🔹 Function to Translate Text
def translate_text():
    source_text = text_input.get("1.0", tk.END).strip()
    src_lang = source_lang.get()
    dest_lang = target_lang.get()

    if not source_text:
        messagebox.showerror("Error", "Please enter text to translate.")
        return

    try:
        response = requests.post(
            url="https://openrouter.ai/api/v1/chat/completions",
            headers={"Authorization": f"Bearer {DEEPSEEK_API_KEY}", "Content-Type": "application/json"},
            data=json.dumps({
                "model": "deepseek/deepseek-r1:free",
                "messages": [
                    {"role": "system", "content": f"Translate the following from {src_lang} to {dest_lang}."},
                    {"role": "user", "content": f"Text: {source_text}"}
                ]
            })
        )

        result = response.json()
        if "choices" in result:
            translated_text = result["choices"][0]["message"]["content"].strip()
            text_output.delete("1.0", tk.END)
            text_output.insert(tk.END, translated_text)
            add_to_history(f"Translated: {source_text[:30]}...", translated_text)  # Store history
        else:
            raise Exception(result.get("error", "Unknown error"))

    except Exception as e:
        messagebox.showerror("Translation Error", str(e))

# 🔹 Function to Add Conversation to History
def add_to_history(query, response):
    chat_history.append((query, response))
    history_listbox.insert(tk.END, query)

# 🔹 Function to Display Selected History
def show_selected_history(event):
    selected_index = history_listbox.curselection()
    if selected_index:
        query, response = chat_history[selected_index[0]]
        text_input.delete("1.0", tk.END)
        text_input.insert(tk.END, query)
        text_output.delete("1.0", tk.END)
        text_output.insert(tk.END, response)

# 🔹 GUI Setup
root = tk.Tk()
root.title("AI Language Translator")
root.geometry("850x650")
root.config(bg="#e3f2fd")

# 🔹 Left Frame - Chat History
history_frame = tk.Frame(root, bg="#B3E5FC", width=250)
history_frame.pack(side=tk.LEFT, fill=tk.Y)

tk.Label(history_frame, text="Chat History", font=("Arial", 14, "bold"), bg="#03A9F4", fg="white").pack(fill=tk.X)
history_listbox = tk.Listbox(history_frame, font=("Arial", 12), height=30, width=30)
history_listbox.pack(padx=10, pady=10, fill=tk.Y, expand=True)
history_listbox.bind("<<ListboxSelect>>", show_selected_history)

# 🔹 Main Content Frame
main_frame = tk.Frame(root, bg="#e3f2fd")
main_frame.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)

tk.Label(main_frame, text="AI Language Translator", font=("Arial", 16, "bold"), bg="#2196F3", fg="white", pady=10).pack(fill=tk.X)

# 🔹 Load & Display LPU Logo
logo_path = "C:/Users/gargk/Downloads/Lovely_Professional_University_logo.png"
if os.path.exists(logo_path):
    try:
        logo_img = Image.open(logo_path).resize((100, 100), Image.Resampling.LANCZOS)
        logo_photo = ImageTk.PhotoImage(logo_img)
        tk.Label(main_frame, image=logo_photo, bg="#e3f2fd").pack(pady=10)
    except Exception:
        tk.Label(main_frame, text="LPU Logo Not Found!", fg="red", font=("Arial", 12, "bold"), bg="#e3f2fd").pack()

# 🔹 Language Selection
frame = tk.Frame(main_frame, bg="#e3f2fd")
frame.pack(pady=10)

tk.Label(frame, text="From:", font=("Arial", 12), bg="#e3f2fd").grid(row=0, column=0, padx=10)
source_lang = ttk.Combobox(frame, values=LANGUAGES, state="readonly", width=20)
source_lang.set("English")
source_lang.grid(row=0, column=1, padx=10)

tk.Label(frame, text="To:", font=("Arial", 12), bg="#e3f2fd").grid(row=1, column=0, padx=10)
target_lang = ttk.Combobox(frame, values=LANGUAGES, state="readonly", width=20)
target_lang.set("Spanish")
target_lang.grid(row=1, column=1, padx=10)

# 🔹 Text Input
tk.Label(main_frame, text="Enter Text:", font=("Arial", 12), bg="#e3f2fd").pack(pady=5)
text_input = tk.Text(main_frame, height=5, width=60, font=("Arial", 12))
text_input.pack(pady=5)

# 🔹 Buttons
btn_frame = tk.Frame(main_frame, bg="#e3f2fd")
btn_frame.pack(pady=5)

tk.Button(btn_frame, text="Enhance Text (AI)", command=enhance_text, font=("Arial", 12, "bold"), bg="#FFC107").grid(row=0, column=0, padx=5)
tk.Button(btn_frame, text="Translate", command=translate_text, font=("Arial", 12, "bold"), bg="#4CAF50", fg="white").grid(row=0, column=1, padx=5)

# 🔹 Translated Text Output
tk.Label(main_frame, text="Translated Text:", font=("Arial", 12), bg="#e3f2fd").pack(pady=5)
text_output = tk.Text(main_frame, height=6, width=60, font=("Arial", 12))
text_output.pack(pady=5)

# 🔹 Run Tkinter Main Loop
root.mainloop()

