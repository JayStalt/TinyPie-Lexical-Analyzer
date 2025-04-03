import tkinter as tk
from tkinter import messagebox, END, BOTTOM
import re

TEST_INPUT = """float mathresult1 = 5*4.3 + 2.1;
float mathresult2 = 4.1 + 2*5.5; 
if (mathresult1 >mathresult2):
	print(“I just built some parse trees” );"""


class LexicalAnalyzer:
    def __init__(self, user_input):
        self.user_input_lines = user_input.split('\n')
        self.current_line_index = 0

    def get_next_line(self):
        if self.current_line_index < len(self.user_input_lines):
            line = self.user_input_lines[self.current_line_index]
            self.current_line_index += 1
            return line
        else:
            return None

    def tokenize_line(self, line):
        # Tokenize the line using the provided token patterns
        token_patterns = [
            (r'"(.*?)"', 'lit_string'),
            (r'\b(if|else|float|int)\b', 'key'),
            (r'(\=|\+|\>|\*)', 'op'),
            (r'(\(|\)|:|"|;)', 'sep'),
            (r'\d+\.\d+', 'lit_float'),
            (r'\b\d+\b', 'lit_int'),
            (r'[a-zA-Z]+[a-zA-Z0-9]*', 'id')
        ]

        tokens = []
        while line:
            line = line.strip()
            for pattern, token_type in token_patterns:
                match = re.match(pattern, line)
                if match:
                    token_value = match.group().strip('"')
                    if token_type == 'lit_string':
                        token_value = token_value.replace('"', '\\"')
                    tokens.append((token_type, token_value))
                    line = line[len(match.group()):].strip()
                    break
            else:
                raise ValueError(f'Invalid token at position: {len(tokens) + 1}')
        return tokens


class TinyPieParser:
    def __init__(self):
        self.parse_tree_debug_output = None
        self.parse_tree = None

    def parse(self, user_input):
        debug_output = "" # Reset the parse tree
        lexer = LexicalAnalyzer(user_input)
        while True:
            line = lexer.get_next_line()
            if not line:
                break
            tokens = lexer.tokenize_line(line)

            # self.parse_tree = self.build_tree(tokens)
            # parse_result = self.parse_line(tokens, lexer.current_line_index)
            # self.parse_tree += f"####Parse tree for line {lexer.current_line_index}####\n{parse_result}\n"

            #####################
            # Now that you have all the tokens in a list, you'll need to parse them out to a tree.
            # This shows you what you need to do:
            # https://ruslanspivak.com/lsbasi-part1/
            ####################

            debug_output = self.build_tree(tokens)

            lexer.current_line_index += 1
        return debug_output

    def build_tree(self, tokens):
        token_counter = 1
        root_token = tokens[0] # This will always be a key
        debug_output = "Building Tree:"

        if len(tokens) <= 1:
            raise Exception("Illegal Line")

        if root_token[0] != "key":
            raise Exception("All lines should start with a key")

        for token in tokens:
            debug_output += f"\n{token[0]}, {token[1]}"

        return debug_output

    def parse_line(self, tokens, line_number):
        first_token_type, first_token_value = tokens[0]
        if first_token_type == 'key':
            if first_token_value == 'if':
                return self.parse_if_statement(tokens)
            if first_token_value == 'float':
                return self.parse_float_value(tokens)

        elif first_token_value == 'print':
            return self.parse_print_statement(tokens)

        elif first_token_type == 'id':
            return self.parse_assignment(tokens)

        return f"Unknown line {line_number}"

    def parse_float_value(self, tokens):
        return f"{tokens[0][0]}, {tokens[0][1]}"

    def parse_assignment(self, tokens):
        variable = tokens[0][1]
        operator = tokens[1][1]
        expression = " ".join(f"{token_type} {token_value}" for token_type,token_value in tokens[2:])
        return f"Assignment: {variable} {operator} {expression}"

    def parse_if_statement(self, tokens):
        condition = " ".join(f"{token_type} {token_value}" for token_type,token_value in tokens[1:])
        return f"If Statement: {condition}"

    def parse_print_statement(self, tokens):
        message = tokens[1][1]
        return f"Print Statement: {message}"


class LexicalAnalyzerGUI:
    def __init__(self, master):
        self.current_processing_line = 0

        ######
        # Root Window
        ######
        self.root = master
        self.root.title("Lexical Analyzer for TinyPie")

        self.master_frame = tk.Frame(self.root)
        self.master_frame.pack()

        ######
        # Top Frame
        ######
        self.top_frame = tk.Frame(self.master_frame)
        self.top_frame.pack(fill=tk.BOTH, side=tk.TOP, expand=True)

        #######
        # Source Code Input
        #######
        self.source_code_frame = tk.Frame(self.top_frame)
        self.source_code_frame.pack(fill=tk.BOTH, side=tk.LEFT, expand=True)

        self.label_input = tk.Label(self.source_code_frame, text="Source Code")
        self.label_input.pack(pady=5, side="top")

        self.user_input_text = tk.Text(self.source_code_frame, width=30, height=15)
        self.user_input_text.pack(fill=tk.BOTH, expand=True)
        self.user_input_text.insert(END, TEST_INPUT)

        self.current_line_label = tk.Label(self.source_code_frame, text=f"Current processing Line: {self.current_processing_line}")
        self.current_line_label.pack(pady=5, side="bottom")


        #######
        # Tokens
        #######
        self.tokens_frame = tk.Frame(self.top_frame)
        self.tokens_frame.pack(fill=tk.BOTH, side=tk.LEFT, expand=True)

        self.label_output = tk.Label(self.tokens_frame, text="Tokens")
        self.label_output.pack(fill=tk.BOTH, pady=5, side="top")

        self.lexical_result_text = tk.Text(self.tokens_frame, width=30, height=5)
        self.lexical_result_text.pack(fill=tk.BOTH, expand=True, side="bottom")
        self.lexical_result_text.config(state=tk.DISABLED)


        #######
        # Parse Tree Text
        #######
        self.parse_tree_frame = tk.Frame(self.top_frame)
        self.parse_tree_frame.pack(fill=tk.BOTH, side=tk.LEFT, expand=True)

        self.label_parse_tree = tk.Label(self.parse_tree_frame, text="Parse Tree")
        self.label_parse_tree.pack(fill=tk.BOTH, pady=5)

        self.parse_tree_text = tk.Text(self.parse_tree_frame, width=50, height=15)
        self.parse_tree_text.pack(fill=tk.BOTH, expand=True)
        self.parse_tree_text.config(state=tk.DISABLED)


        #######
        # Buttons
        #######

        self.bottom_frame=tk.Frame(self.master_frame)
        self.bottom_frame.pack(fill=tk.BOTH, side=tk.LEFT, expand=True)

        self.next_button = tk.Button(self.bottom_frame, text="Next Line", command=self.process_next_line)
        self.next_button.pack(side=tk.RIGHT, padx=10, pady=10, anchor=tk.SE)

        self.quit_button = tk.Button(self.bottom_frame, text="Quit", command=self.root.quit)
        self.quit_button.pack(side=tk.RIGHT, padx=10, pady=10, anchor=tk.SE)

        self.lexical_analyzer = None
        self.tiny_pie_parser = None

    def process_next_line(self):
        if self.lexical_analyzer is None:
            user_input = self.user_input_text.get("1.0", tk.END)
            self.lexical_analyzer = LexicalAnalyzer(user_input)
            self.tiny_pie_parser = TinyPieParser()
        line = self.lexical_analyzer.get_next_line()
        if line:
            self.current_processing_line += 1
            self.current_line_label.config(text=f"Current Processing Line:{self.current_processing_line}")
            self.display_tokenized_line(line)
            self.display_parse_tree(self.tiny_pie_parser.parse(line))
        else:
            messagebox.showinfo("End of Input", "No more lines to process.")

    def display_tokenized_line(self, line):
        try:
            tokens = self.lexical_analyzer.tokenize_line(line)
            self.lexical_result_text.config(state=tk.NORMAL)
            self.lexical_result_text.delete(1.0, tk.END)
            self.lexical_result_text.insert(tk.END, f"Tokens for Line{self.current_processing_line}:\n")
            self.lexical_result_text.insert(tk.END, "\n".join([f"<{token_type},{token_value}>" for token_type, token_value in tokens]) + "\n\n")
            self.lexical_result_text.config(state=tk.DISABLED)
        except ValueError as e:
            print(f"Error: {e}")

    def display_parse_tree(self, parse_tree):
        if "Invalid token" in parse_tree:
            messagebox.showerror("Invalid Token", parse_tree)
        else:
            self.parse_tree_text.config(state=tk.NORMAL)
            self.parse_tree_text.delete(1.0, tk.END)
            self.parse_tree_text.insert(tk.END, parse_tree)
            self.parse_tree_text.config(state=tk.DISABLED)


if __name__ == '__main__':
    root = tk.Tk()
    lexical_analyzer_gui = LexicalAnalyzerGUI(root)
    root.mainloop()
