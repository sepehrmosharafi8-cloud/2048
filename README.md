# 2048
import tkinter as tk
import random
import os
import winsound

SIZE = 4
FONT = ("Arial", 24, "bold")
SCORE_FONT = ("Arial", 14, "bold")
HIGH_SCORE_FILE = "highscore.txt"

COLORS = {
    0:    ("#cdc1b4", "#776e65"),
    2:    ("#eee4da", "#776e65"),
    4:    ("#ede0c8", "#776e65"),
    8:    ("#f2b179", "#f9f6f2"),
    16:   ("#f59563", "#f9f6f2"),
    32:   ("#f67c5f", "#f9f6f2"),
    64:   ("#f65e3b", "#f9f6f2"),
    128:  ("#edcf72", "#f9f6f2"),
    256:  ("#edcc61", "#f9f6f2"),
    512:  ("#edc850", "#f9f6f2"),
    1024: ("#edc53f", "#f9f6f2"),
    2048: ("#edc22e", "#f9f6f2"),
}

SOUNDS = {
    4: 400,
    8: 500,
    16: 600,
    32: 700,
    64: 800,
    128: 900,
    256: 1000,
    512: 1100,
    1024: 1200,
    2048: 1400,
}

class Game2048:
    def __init__(self, root):
        self.root = root
        self.root.title("2048")

        self.score = 0
        self.high_score = self.load_high_score()
        self.board = [[0]*SIZE for _ in range(SIZE)]

        top = tk.Frame(root)
        top.pack()

        self.score_label = tk.Label(top, text="Score: 0", font=SCORE_FONT)
        self.score_label.pack(side="left", padx=10)

        self.high_label = tk.Label(top, text=f"Best: {self.high_score}", font=SCORE_FONT)
        self.high_label.pack(side="right", padx=10)

        self.frame = tk.Frame(root, bg="#bbada0")
        self.frame.pack(padx=10, pady=10)

        self.cells = [[None]*SIZE for _ in range(SIZE)]
        for r in range(SIZE):
            for c in range(SIZE):
                lbl = tk.Label(self.frame, width=4, height=2, font=FONT)
                lbl.grid(row=r, column=c, padx=5, pady=5)
                self.cells[r][c] = lbl

        self.add_number()
        self.add_number()
        self.update_ui()

        root.bind("<Key>", self.key_pressed)

    def load_high_score(self):
        if os.path.exists(HIGH_SCORE_FILE):
            return int(open(HIGH_SCORE_FILE).read())
        return 0

    def save_high_score(self):
        with open(HIGH_SCORE_FILE, "w") as f:
            f.write(str(self.high_score))

    def max_tile(self):
        return max(max(row) for row in self.board)

    def play_sound(self, value):
        freq = SOUNDS.get(value, 800)
        winsound.Beep(freq, 120)

    def add_number(self):
        empty = [(r, c) for r in range(SIZE)
                        for c in range(SIZE)
                        if self.board[r][c] == 0]
        if not empty:
            return

        r, c = random.choice(empty)
        max_value = self.max_tile()

        if max_value >= 256:
            new_value = 8
        elif max_value >= 64:
            new_value = 4
        else:
            new_value = 2 if random.random() < 0.9 else 4

        self.board[r][c] = new_value

    def slide(self, row):
        row = [x for x in row if x != 0]
        for i in range(len(row) - 1):
            if row[i] == row[i + 1]:
                row[i] *= 2
                self.score += row[i]
                self.play_sound(row[i])
                row[i + 1] = 0
        row = [x for x in row if x != 0]
        return row + [0] * (SIZE - len(row))

    def move_left(self):
        self.board = [self.slide(row) for row in self.board]

    def move_right(self):
        self.board = [self.slide(row[::-1])[::-1] for row in self.board]

    def move_up(self):
        for c in range(SIZE):
            col = [self.board[r][c] for r in range(SIZE)]
            col = self.slide(col)
            for r in range(SIZE):
                self.board[r][c] = col[r]

    def move_down(self):
        for c in range(SIZE):
            col = [self.board[r][c] for r in range(SIZE)][::-1]
            col = self.slide(col)[::-1]
            for r in range(SIZE):
                self.board[r][c] = col[r]

    def can_move(self):
        for r in range(SIZE):
            for c in range(SIZE):
                if self.board[r][c] == 0:
                    return True
                if c < SIZE - 1 and self.board[r][c] == self.board[r][c + 1]:
                    return True
                if r < SIZE - 1 and self.board[r][c] == self.board[r + 1][c]:
                    return True
        return False

    def check_win(self):
        return any(2048 in row for row in self.board)

    def key_pressed(self, event):
        old = [row[:] for row in self.board]

        if event.keysym == "Left":
            self.move_left()
        elif event.keysym == "Right":
            self.move_right()
        elif event.keysym == "Up":
            self.move_up()
        elif event.keysym == "Down":
            self.move_down()

        if old != self.board:
            self.add_number()
            self.update_ui()

            if self.check_win():
                self.end_game("You Win")
            elif not self.can_move():
                self.end_game("Game Over")

    def end_game(self, msg):
        if self.score > self.high_score:
            self.high_score = self.score
            self.save_high_score()

        popup = tk.Toplevel()
        popup.title("Game End")
        tk.Label(popup, text=msg, font=("Arial", 20, "bold")).pack(pady=10)
        tk.Label(popup, text=f"Score: {self.score}").pack()
        tk.Button(popup, text="Exit", command=self.root.destroy).pack(pady=10)

    def update_ui(self):
        self.score_label.config(text=f"Score: {self.score}")

        if self.score > self.high_score:
            self.high_score = self.score
            self.high_label.config(text=f"Best: {self.high_score}")
            self.save_high_score()

        for r in range(SIZE):
            for c in range(SIZE):
                v = self.board[r][c]
                bg, fg = COLORS.get(v, COLORS[2048])
                self.cells[r][c].config(
                    text="" if v == 0 else v,
                    bg=bg,
                    fg=fg
                )

if __name__ == "__main__":
    root = tk.Tk()
    Game2048(root)
    root.mainloop()
