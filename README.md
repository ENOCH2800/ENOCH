# ENOCH
import openai
import os
import tkinter as tk
from tkinter import scrolledtext, messagebox


# --- OpenAI API Setup ---
openai.api_key = os.environ.get("OPENAI_API_KEY")  # Use environment variable for API key
if not openai.api_key:
    messagebox.showerror("Error", "OpenAI API key not found. Please set the OPENAI_API_KEY environment variable.")
    exit()


# --- Chat History Management ---
class ChatHistory:
    """Class to manage chat history."""

    def __init__(self):
        self.history = []

    def initialize_chat_history(self):
        """Initializes an empty chat history."""
        self.history = []

    def add_message(self, role, content):
        """Adds a message to the chat history.

        Args:
            role: The role of the message (e.g., "system", "user", "assistant").
            content: The content of the message.
        """
        self.history.append({"role": role, "content": content})

    def get_formatted_history(self):
        """Formats the chat history for OpenAI API."""
        return [{"role": msg["role"], "content": msg["content"]} for msg in self.history]

    def clear_history(self):
        """Clears the chat history."""
        self.initialize_chat_history()


# --- OpenAI API Interaction ---
class OpenAIChatbot:
    """Class to interact with OpenAI API."""

    def __init__(self, model="gpt-3.5-turbo"):
        self.model = model
        self.chat_history = ChatHistory()

    def get_completion(self, prompt):
        """Sends a prompt to the OpenAI API and receives a response.

        Args:
            prompt: The user's prompt.

        Returns:
            The assistant's response, or None if there's an error.
        """
        self.chat_history.add_message("user", prompt)

        try:
            response = openai.ChatCompletion.create(
                model=self.model,
                messages=[
                    {"role": "system", "content": "You are a helpful assistant."},
                    *self.chat_history.get_formatted_history(),
                ],
                temperature=0.7,
                max_tokens=150,
            )

            completion = response.choices[0].message["content"].strip()
            self.chat_history.add_message("assistant", completion)
            return completion

        except openai.error.OpenAIError as e:
            return f"Error interacting with OpenAI API: {e}"


# --- GUI Implementation ---
class ChatbotGUI:
    """Class to create and manage the chatbot GUI."""

    def __init__(self, root):
        self.root = root
        self.root.title("ChatGPT-like Assistant")

        # Initialize chatbot
        self.chatbot = OpenAIChatbot()

        # Create UI components
        self.chat_display = scrolledtext.ScrolledText(root, wrap=tk.WORD, state="disabled", height=20, width=80)
        self.chat_display.grid(row=0, column=0, padx=10, pady=10, columnspan=2)

        self.user_input = tk.Entry(root, width=70)
        self.user_input.grid(row=1, column=0, padx=10, pady=10, sticky="w")

        self.send_button = tk.Button(root, text="Send", command=self.send_message)
        self.send_button.grid(row=1, column=1, padx=10, pady=10, sticky="e")

        self.clear_button = tk.Button(root, text="Clear Chat", command=self.clear_chat)
        self.clear_button.grid(row=2, column=0, padx=10, pady=10, sticky="w")

        self.exit_button = tk.Button(root, text="Exit", command=self.exit_application)
        self.exit_button.grid(row=2, column=1, padx=10, pady=10, sticky="e")

    def display_message(self, role, message):
        """Displays a message in the chat display.

        Args:
            role: The role of the message sender (e.g., "User", "Assistant").
            message: The message content.
        """
        self.chat_display.config(state="normal")
        self.chat_display.insert(tk.END, f"{role}: {message}\n\n")
        self.chat_display.yview(tk.END)
        self.chat_display.config(state="disabled")

    def send_message(self):
        """Handles sending user input to the chatbot."""
        user_input = self.user_input.get().strip()

        if not user_input:
            return

        # Display user message
        self.display_message("You", user_input)
        self.user_input.delete(0, tk.END)

        # Get chatbot response
        response = self.chatbot.get_completion(user_input)
        self.display_message("Assistant", response)

    def clear_chat(self):
        """Clears the chat display and history."""
        self.chatbot.chat_history.clear_history()
        self.chat_display.config(state="normal")
        self.chat_display.delete(1.0, tk.END)
        self.chat_display.config(state="disabled")

    def exit_application(self):
        """Exits the application."""
        self.root.quit()


# --- Main Application ---
if __name__ == "__main__":
    root = tk.Tk()
    app = ChatbotGUI(root)
    root.mainloop()