# Typing-Speed-Test-
import tkinter as tk
import time
import random

# ---------------- Main Window ----------------
root = tk.Tk()
root.title("Typing Speed Test")
root.geometry("800x500")

# Dark theme colors
BG_COLOR = "#000000"
CARD_COLOR = "#121212"
TEXT_BG = "#1e1e1e"
TEXT_FG = "#ffffff"
ACCENT_GREEN = "#00c853"
ACCENT_RED = "#ff5252"
ACCENT_BLUE = "#2962ff"
ACCENT_YELLOW = "#ffd600"

root.configure(bg=BG_COLOR)

# ---------------- Paragraphs ----------------
paragraphs = [
    "Python is an easy to learn, powerful programming language. It has efficient high level data structures and a simple but effective approach to object oriented programming.",
    "Practice makes a person perfect. The more you type every day, the better your speed and accuracy will become over time.",
    "Programming teaches you how to think logically. It helps you break big problems into smaller, manageable steps.",
    "Good code is not only correct, but also readable and easy to understand for other people working on the same project.",
    "Debugging is an essential skill for every programmer. It helps you find and fix errors in your code efficiently.",
    "Consistent effort over a long period of time is more powerful than a single burst of energy and motivation.",
    "Learning new skills can feel difficult at first, but small daily progress leads to big improvements in the long run.",
    "Reading other people's code is a great way to learn new techniques and improve your own coding style.",
    "Technology changes quickly, so a good developer must always be curious and willing to learn new tools.",
    "Writing comments in your code can save you and your teammates a lot of time and confusion in the future.",
    "Typing speed tests help you measure your ability to type quickly and accurately under time pressure.",
    "Focus on typing without looking at the keyboard. This habit will greatly improve your typing speed.",
    "Short breaks during long study sessions help your brain stay fresh and focused on the task.",
    "Setting clear goals before you start working makes it easier to track your progress and stay motivated.",
    "Learning to code opens up opportunities in many fields such as web development, data science and automation.",
    "Accuracy is more important than speed when you are just starting. Speed will naturally increase with practice.",
    "A growth mindset means believing that your abilities can improve with effort and good strategies.",
    "Making mistakes is a natural part of learning. Each error shows you where you can improve next time.",
    "Time management is one of the most valuable skills you can develop as a student and a professional.",
    "Typing regularly for a few minutes every day is better than practicing for long hours only once a week."
]

text_to_type = random.choice(paragraphs)

# ---------------- Variables ----------------
start_time = 0
running = False
time_limit = 30  # seconds

# ---------------- Functions ----------------
def choose_new_paragraph():
    """Pick a new random paragraph and reset the display."""
    global text_to_type, running
    running = False

    # pick a new random paragraph
    new_para = text_to_type
    while new_para == text_to_type:  # try to avoid same one in a row
        new_para = random.choice(paragraphs)
    text_to_type = new_para

    paragraph_label.config(text=text_to_type)

    # reset typing area and labels
    text_box.config(state="normal")
    text_box.delete("1.0", tk.END)
    text_box.tag_remove("correct", "1.0", tk.END)
    text_box.tag_remove("wrong", "1.0", tk.END)
    text_box.config(state="disabled")

    timer_label.config(text=f"Time Left: {time_limit}s")
    wpm_label.config(text="WPM: 0")
    result_label.config(text="")


def start_test():
    """Start timing and enable typing."""
    global start_time, running
    running = True
    start_time = time.time()

    text_box.config(state="normal")
    text_box.delete("1.0", tk.END)
    text_box.tag_remove("correct", "1.0", tk.END)
    text_box.tag_remove("wrong", "1.0", tk.END)
    text_box.focus()

    timer_label.config(text=f"Time Left: {time_limit}s")
    wpm_label.config(text="WPM: 0")
    result_label.config(text="")

    update_timer()


def get_typed_text():
    return text_box.get("1.0", "end-1c")


def highlight_text():
    """Highlight correct and wrong letters in the Text widget."""
    typed = get_typed_text()
    original = text_to_type

    text_box.tag_remove("correct", "1.0", tk.END)
    text_box.tag_remove("wrong", "1.0", tk.END)

    for i, ch in enumerate(typed):
        idx_start = f"1.0+{i}c"
        idx_end = f"1.0+{i+1}c"
        if i < len(original) and ch == original[i]:
            text_box.tag_add("correct", idx_start, idx_end)
        else:
            text_box.tag_add("wrong", idx_start, idx_end)


def count_correct_words(typed):
    """Count correctly typed words in correct positions."""
    original_words = text_to_type.split()
    typed_words = typed.split()
    correct = 0
    for i in range(min(len(original_words), len(typed_words))):
        if typed_words[i] == original_words[i]:
            correct += 1
    return correct, len(original_words)


def update_live_wpm():
    """Update WPM label while typing."""
    if not running:
        return
    typed = get_typed_text()
    elapsed = time.time() - start_time
    if elapsed <= 0:
        elapsed = 0.1
    correct_words, total_words = count_correct_words(typed)
    wpm = round((correct_words / elapsed) * 60)
    wpm_label.config(text=f"WPM: {wpm}")


def on_key_release(event=None):
    """Handle typing events: highlight, live WPM, check completion."""
    if not running:
        return
    highlight_text()
    update_live_wpm()
    check_text_finished()


def check_text_finished():
    """Check if user finished typing the paragraph."""
    if not running:
        return
    typed = get_typed_text()
    if typed == text_to_type:
        finish_test(time_up=False)


def update_timer():
    """Update timer every second."""
    global running
    if not running:
        return

    elapsed = time.time() - start_time
    remaining = time_limit - int(elapsed)

    if remaining <= 0:
        finish_test(time_up=True)
    else:
        timer_label.config(text=f"Time Left: {remaining}s")
        root.after(1000, update_timer)


def save_results_to_file(total_time, wpm, accuracy, time_up):
    """Save result line to a text file."""
    status = "Time Up" if time_up else "Completed"
    timestamp = time.strftime("%Y-%m-%d %H:%M:%S")
    line = f"{timestamp} | {status} | Time: {total_time:.2f}s | WPM: {wpm} | Accuracy: {accuracy}%\n"
    with open("typing_results.txt", "a", encoding="utf-8") as f:
        f.write(line)


def finish_test(time_up=False):
    """End the test and calculate results."""
    global running
    if not running:
        return
    running = False

    end_time = time.time()
    total_time = round(end_time - start_time, 2)
    if total_time <= 0:
        total_time = 0.1

    typed = get_typed_text()
    highlight_text()  # make sure final highlight is correct

    correct_words, total_words = count_correct_words(typed)
    wpm = round((correct_words / total_time) * 60)
    accuracy = round((correct_words / total_words) * 100, 2)

    text_box.config(state="disabled")
    wpm_label.config(text=f"WPM: {wpm}")

    if time_up:
        result_label.config(
            text=f"⏰ Time's up!\nSpeed: {wpm} WPM\nAccuracy: {accuracy}%"
        )
    else:
        result_label.config(
            text=f"✅ Done!\nTime: {total_time}s\nSpeed: {wpm} WPM\nAccuracy: {accuracy}%"
        )

    save_results_to_file(total_time, wpm, accuracy, time_up)


# ---------------- GUI Layout ----------------
title_label = tk.Label(
    root,
    text="Typing Speed Test",
    font=("Arial", 22, "bold"),
    bg=BG_COLOR,
    fg=ACCENT_YELLOW
)
title_label.pack(pady=10)

frame = tk.Frame(root, bg=CARD_COLOR, padx=15, pady=15)
frame.pack(pady=10, padx=10, fill="both", expand=True)

paragraph_title = tk.Label(
    frame,
    text="Text to Type:",
    font=("Arial", 12, "bold"),
    bg=CARD_COLOR,
    fg=ACCENT_GREEN
)
paragraph_title.pack(anchor="w")

paragraph_label = tk.Label(
    frame,
    text=text_to_type,
    wraplength=750,
    justify="left",
    font=("Arial", 12),
    bg=CARD_COLOR,
    fg=TEXT_FG
)
paragraph_label.pack(pady=5, anchor="w")

typing_label = tk.Label(
    frame,
    text="Type here:",
    font=("Arial", 12, "bold"),
    bg=CARD_COLOR,
    fg=ACCENT_BLUE
)
typing_label.pack(anchor="w", pady=(10, 0))

text_box = tk.Text(
    frame,
    height=4,
    width=80,
    font=("Consolas", 12),
    bg=TEXT_BG,
    fg=TEXT_FG,
    insertbackground=TEXT_FG,
    wrap="word",
    relief="flat",
    padx=8,
    pady=8
)
text_box.pack(pady=5, fill="x")
text_box.bind("<KeyRelease>", on_key_release)
text_box.config(state="disabled")

# Tags for highlighting
text_box.tag_config("correct", foreground=ACCENT_GREEN)
text_box.tag_config("wrong", foreground=ACCENT_RED)

# Buttons
buttons_frame = tk.Frame(frame, bg=CARD_COLOR)
buttons_frame.pack(pady=10)

start_button = tk.Button(
    buttons_frame,
    text="Start Test",
    command=start_test,
    bg=ACCENT_GREEN,
    fg="black",
    activebackground="#00e676",
    width=14,
    font=("Arial", 11, "bold"),
    relief="flat"
)
start_button.grid(row=0, column=0, padx=5)

new_button = tk.Button(
    buttons_frame,
    text="New Statement",
    command=choose_new_paragraph,
    bg=ACCENT_BLUE,
    fg="white",
    activebackground="#448aff",
    width=14,
    font=("Arial", 11, "bold"),
    relief="flat"
)
new_button.grid(row=0, column=1, padx=5)

# Info labels
info_frame = tk.Frame(frame, bg=CARD_COLOR)
info_frame.pack(pady=5, fill="x")

timer_label = tk.Label(
    info_frame,
    text=f"Time Left: {time_limit}s",
    font=("Arial", 12, "bold"),
    bg=CARD_COLOR,
    fg=ACCENT_RED
)
timer_label.pack(side="left")

wpm_label = tk.Label(
    info_frame,
    text="WPM: 0",
    font=("Arial", 12, "bold"),
    bg=CARD_COLOR,
    fg=ACCENT_YELLOW
)
wpm_label.pack(side="right")

result_label = tk.Label(
    root,
    text="",
    font=("Arial", 13, "bold"),
    bg=BG_COLOR,
    fg=TEXT_FG,
    justify="center"
)
result_label.pack(pady=10)

# Start with a random paragraph ready
choose_new_paragraph()

root.mainloop()
