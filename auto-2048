import random
import math

def init_board():
    """Initialize the 4x4 game board with two starting tiles."""
    board = [[0] * 4 for _ in range(4)]
    add_random_tile(board)
    add_random_tile(board)
    return board

def add_random_tile(board):
    """Add a 2 or 4 to a random empty spot on the board."""
    empty = [(r, c) for r in range(4) for c in range(4) if board[r][c] == 0]
    if empty:
        r, c = random.choice(empty)
        board[r][c] = 2 if random.random() < 0.9 else 4

def print_board(board):
    """Print the game board."""
    for row in board:
        print(" ".join(f"{cell:4}" if cell != 0 else "   *" for cell in row))
    print()

def move_row_left(row):
    """Slide and merge a single row to the left."""
    new_row = [cell for cell in row if cell != 0]
    merged = []
    while new_row:
        if len(new_row) > 1 and new_row[0] == new_row[1]:
            merged.append(new_row[0] * 2)
            new_row = new_row[2:]
        else:
            merged.append(new_row[0])
            new_row = new_row[1:]
    return merged + [0] * (4 - len(merged))

def rotate_board_clockwise(board):
    """Rotate the board 90 degrees clockwise."""
    return [list(row) for row in zip(*board[::-1])]

def rotate_board_counterclockwise(board):
    """Rotate the board 90 degrees counterclockwise."""
    return [list(row) for row in zip(*board)][::-1]

def left(board):
    """Move tiles left."""
    moved = False
    for r in range(4):
        new_row = move_row_left(board[r])
        if new_row != board[r]:
            moved = True
        board[r] = new_row
    return moved

def right(board):
    """Move tiles right."""
    for r in range(4):
        board[r] = board[r][::-1]
    moved = left(board)
    for r in range(4):
        board[r] = board[r][::-1]
    return moved

def up(board):
    """Move tiles up."""
    rotated = rotate_board_counterclockwise(board)
    moved = left(rotated)
    board[:] = rotate_board_clockwise(rotated)
    return moved

def down(board):
    """Move tiles down."""
    rotated = rotate_board_clockwise(board)
    moved = left(rotated)
    board[:] = rotate_board_counterclockwise(rotated)
    return moved

def is_game_over(board):
    """Check if no moves are possible."""
    for r in range(4):
        for c in range(4):
            if board[r][c] == 0:
                return False
            if r < 3 and board[r][c] == board[r + 1][c]:
                return False
            if c < 3 and board[r][c] == board[r][c + 1]:
                return False
    return True

def check_2048(board):
    """Check if the 2048 tile exists on the board."""
    for row in board:
        if 2048 in row:
            return True
    return False

def normalize(x):
    return x / 2048 if x else 0

def create_points(board):
    """Create a scoring matrix based on tile values and positions."""
    board_p = [[0] * 4 for _ in range(4)]
    for r in range(4):
        for c in range(4):
            if board[r][c] != 0:
                left = board[r][c - 1] if c > 0 else 0
                right = board[r][c + 1] if c < 3 else 0
                up = board[r + 1][c] if r < 3 else 0
                down = board[r - 1][c] if r < 3 else 0
                if left < board[r][c] or right > board[r][c] or up <= board[r][c] or down >= board[r][c]:
                    board_p[r][c] = normalize(board[r][c])
                if left < board[r][c] and right > board[r][c]:
                    board_p[r][c] = normalize(2 * board[r][c])
                if left < board[r][c] and right > board[r][c] and up <= board[r][c] and down >= board[r][c]:
                    board_p[r][c] = normalize(3 * board[r][c])
                if left < board[r][c] and right > board[r][c] and up == board[r][c] and down == board[r][c]:
                    board_p[r][c] = normalize(6 * board[r][c])
    return board_p

def get_points(board):
    """Calculate the total score of the board."""
    
    zeros = ((r, c) for r in range(4) for c in range(4) if board[r][c] == 0)
    twos = ((r, c) for r in range(4) for c in range(4) if board[r][c] == 2)
    length_of_zeros = sum(1 for _ in zeros)
    length_of_twos = sum(1 for _ in twos)
    points = 0
    board_p = create_points(board)
    for row in board_p:
        points += sum(row) * ((length_of_zeros + 0.01) / (length_of_twos + 0.01) / 10) 
    return points

def evaluate_future_states(board, depth=1):
    """
    Evaluate the board state by simulating moves up to a specified depth.

    Args:
        board: The current game board.
        depth: How far to look ahead (1 for next best, 2 for next-next best).

    Returns:
        A dictionary mapping moves ('w', 'a', 's', 'd') to their cumulative scores,
        or a single float score when depth reaches 0.
    """
    if depth == 0:
        return get_points(board)  # Base case: directly return the board's score as a float.

    possible_moves = {
        'w': [row[:] for row in board],
        'a': [row[:] for row in board],
        's': [row[:] for row in board],
        'd': [row[:] for row in board],
    }
    move_scores = {}

    for move, board_copy in possible_moves.items():
        if move == 'w' and up(board_copy):
            move_scores[move] = get_points(board_copy) + (
                max(evaluate_future_states(board_copy, depth - 1).values())
                if depth > 1 else 0
            )
        elif move == 'a' and left(board_copy):
            move_scores[move] = get_points(board_copy) + (
                max(evaluate_future_states(board_copy, depth - 1).values())
                if depth > 1 else 0
            )
        elif move == 's' and down(board_copy):
            move_scores[move] = get_points(board_copy) + (
                max(evaluate_future_states(board_copy, depth - 1).values())
                if depth > 1 else 0
            )
        elif move == 'd' and right(board_copy):
            move_scores[move] = get_points(board_copy) + (
                max(evaluate_future_states(board_copy, depth - 1).values())
                if depth > 1 else 0
            )

    return move_scores


def evaluate_moves_with_lookahead(board, lookahead=1):
    """
    Evaluate all possible moves with a specified lookahead depth.

    Args:
        board: The current game board.
        lookahead: How far to look ahead.

    Returns:
        A dictionary mapping moves ('w', 'a', 's', 'd') to their cumulative scores.
    """
    return evaluate_future_states(board, depth=lookahead)


def make_best_move_with_lookahead(board, lookahead=1):
    """
    Make the best possible move based on scores with a specified lookahead depth.

    Args:
        board: The current game board.
        lookahead: How far to look ahead.

    Returns:
        True if a move was made, False otherwise.
    """
    move_scores = evaluate_moves_with_lookahead(board, lookahead)
    if not move_scores:
        print("No valid moves available.")
        return False

    # Sort moves by scores in descending order
    sorted_moves = sorted(move_scores.items(), key=lambda x: x[1], reverse=True)

    for move, score in sorted_moves:
        if move == 'w' and up(board):
            print(f"Making move: {move} (Score: {score}, Lookahead: {lookahead})")
            return True
        elif move == 'a' and left(board):
            print(f"Making move: {move} (Score: {score}, Lookahead: {lookahead})")
            return True
        elif move == 's' and down(board):
            print(f"Making move: {move} (Score: {score}, Lookahead: {lookahead})")
            return True
        elif move == 'd' and right(board):
            print(f"Making move: {move} (Score: {score}, Lookahead: {lookahead})")
            return True

    print("No valid moves could be executed.")
    return False

def main():
    board = init_board()
    print_board(board)

    while True:
        print_board(board)
        # Make the best move based on points
        if not make_best_move_with_lookahead(board,12):
            if is_game_over(board):
                print("Game over!")
                break
        else:
            if check_2048(board):
                print_board(board)
                print("Congratulations! You've reached 4096!")
                break
            add_random_tile(board)

if __name__ == "__main__":
    main()
