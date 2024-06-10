import streamlit as st
import numpy as np
import matplotlib.pyplot as plt
import io
from PIL import Image
import cv2

def create_sudoku_board():
    base = 3
    side = base * base

    def pattern(r, c):
        return (base * (r % base) + r // base + c) % side

    def shuffle(s):
        return np.random.permutation(s)

    r_base = range(base)
    rows = [g * base + r for g in shuffle(r_base) for r in shuffle(r_base)]
    cols = [g * base + c for g in shuffle(r_base) for c in shuffle(r_base)]
    nums = shuffle(range(1, base * base + 1))

    board = np.array([[nums[pattern(r, c)] for c in cols] for r in rows])

    squares = side * side
    empties = squares * 3 // 4
    for p in np.random.choice(range(squares), empties, replace=False):
        board[p // side][p % side] = 0

    return board

def is_possible(board, row, col, num):
    for i in range(9):
        if board[row][i] == num or board[i][col] == num:
            return False

    start_row, start_col = 3 * (row // 3), 3 * (col // 3)
    for i in range(3):
        for j in range(3):
            if board[start_row + i][start_col + j] == num:
                return False

    return True

def solve_sudoku(board):
    for row in range(9):
        for col in range(9):
            if board[row][col] == 0:
                for num in range(1, 10):
                    if is_possible(board, row, col, num):
                        board[row][col] = num
                        if solve_sudoku(board):
                            return True
                        board[row][col] = 0
                return False
    return True

def plot_sudoku(board, title="Sudoku"):
    fig, ax = plt.subplots(figsize=(6, 6))
    for i in range(10):
        ax.plot([i, i], [0, 9], color="black")
        ax.plot([0, 9], [i, i], color="black")

    for i in range(9):
        for j in range(9):
            if board[i][j] != 0:
                ax.text(j + 0.5, 8.5 - i, board[i][j], va='center', ha='center')

    ax.set_xticks([])
    ax.set_yticks([])
    ax.set_title(title)
    st.pyplot(fig)

    buf = io.BytesIO()
    plt.savefig(buf, format='png')
    buf.seek(0)
    return buf

def main():
    st.title("Sudoku Oluşturma ve Çözme Uygulaması")

    if st.button("Yeni Sudoku Oluştur"):
        board = create_sudoku_board()
        solved_board = board.copy()  # Make a copy to reveal the answer
        solve_sudoku(solved_board)  # Solve the Sudoku puzzle
        st.write("Yeni Sudoku Tahtası:")
        st.write(board)
        st.write("Çözülen Sudoku Tahtası:")  # Show the solved puzzle
        st.write(solved_board)
        buf = plot_sudoku(board, "Yeni Sudoku")
        st.download_button("Sudoku'yu İndir (PNG)", buf, "sudoku.png")

    uploaded_file = st.file_uploader("Bir Sudoku Dosyası Yükleyin (.npy formatında)", type="npy")

    if uploaded_file is not None:
        sudoku_array = np.load(uploaded_file)
        st.write("Yüklenen Sudoku Tahtası:")
        st.write(sudoku_array)
        solved_board = sudoku_array.copy()  # Make a copy to reveal the answer
        solve_sudoku(solved_board)  # Solve the Sudoku puzzle
        st.write("Çözülen Sudoku Tahtası:")  # Show the solved puzzle
        st.write(solved_board)

if __name__ == "__main__":
    main()
