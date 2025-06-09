# Minesweeper-game
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <string.h>

#define MAX_SIZE 10

char board[MAX_SIZE][MAX_SIZE];       // Player's view
char mine_map[MAX_SIZE][MAX_SIZE];    // Real mine locations
int rows, cols, mines;
int revealed_cells = 0;
int flag_count = 0;

void init_board() {
    for (int i = 0; i < rows; i++)
        for (int j = 0; j < cols; j++) {
            board[i][j] = '*';
            mine_map[i][j] = '0';
        }
}

void place_mines() {
    int placed = 0;
    while (placed < mines) {
        int r = rand() % rows;
        int c = rand() % cols;
        if (mine_map[r][c] != 'M') {
            mine_map[r][c] = 'M';
            placed++;
        }
    }
}

int count_adjacent_mines(int r, int c) {
    int count = 0;
    for (int dr = -1; dr <= 1; dr++)
        for (int dc = -1; dc <= 1; dc++) {
            int nr = r + dr, nc = c + dc;
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && mine_map[nr][nc] == 'M')
                count++;
        }
    return count;
}

void reveal_cell(int r, int c) {
    if (r < 0 || r >= rows || c < 0 || c >= cols || board[r][c] != '*')
        return;

    int count = count_adjacent_mines(r, c);
    board[r][c] = (count == 0) ? ' ' : count + '0';
    revealed_cells++;

    if (count == 0) {
        for (int dr = -1; dr <= 1; dr++)
            for (int dc = -1; dc <= 1; dc++)
                if (dr != 0 || dc != 0)
                    reveal_cell(r + dr, c + dc);
    }
}

void print_board() {
    printf("\n  Remaining Mines: %d\n\n", mines - flag_count);

    // Print column header
    printf("    ");
    for (int j = 0; j < cols; j++)
        printf(" %2d", j);
    printf("\n");

    // Print top border
    printf("   +");
    for (int j = 0; j < cols; j++) {
        printf("---");
        if (j == cols - 1) printf("+");
    }
    printf("\n");

    // Print rows
    for (int i = 0; i < rows; i++) {
        printf("%2d |", i);
        for (int j = 0; j < cols; j++) {
            char display;
            switch (board[i][j]) {
                case '*': display = '#'; break;   // unrevealed
                case 'F': display = 'F'; break;   // flag
                case ' ': display = ' '; break;   // empty
                default:  display = board[i][j]; break; // number or mine
            }
            printf(" %c ", display);
        }
        printf("|\n");

        // Print row separator
        printf("   +");
        for (int j = 0; j < cols; j++) {
            printf("---");
            if (j == cols - 1) printf("+");
        }
        printf("\n");
    }
}

int check_win() {
    return (rows * cols - revealed_cells == mines);
}

void toggle_flag(int r, int c) {
    if (r < 0 || r >= rows || c < 0 || c >= cols) {
        printf("Coordinates out of bounds!\n");
        return;
    }

    if (board[r][c] == '*') {
        board[r][c] = 'F';
        flag_count++;
    } else if (board[r][c] == 'F') {
        board[r][c] = '*';
        flag_count--;
    } else {
        printf("This cell has already been revealed. You cannot place a flag.\n");
    }
}

int main() {
    srand(time(NULL));

    printf("Enter board size (max %d x %d): ", MAX_SIZE, MAX_SIZE);
    scanf("%d %d", &rows, &cols);
    if (rows > MAX_SIZE || cols > MAX_SIZE || rows <= 0 || cols <= 0) {
        printf("Invalid board size!\n");
        return 1;
    }

    printf("Enter number of mines: ");
    scanf("%d", &mines);
    if (mines >= rows * cols) {
        printf("Too many mines!\n");
        return 1;
    }

    init_board();
    place_mines();

    while (1) {
        print_board();
        printf("Command (F row col = flag/unflag, O row col = open, or row col = open): ");

        char cmd[10];
        int r, c;
        scanf("%s", cmd);

        if (cmd[0] == 'F' || cmd[0] == 'f') {
            scanf("%d %d", &r, &c);
            toggle_flag(r, c);
        } else if (cmd[0] == 'O' || cmd[0] == 'o') {
            scanf("%d %d", &r, &c);
            if (board[r][c] == 'F') {
                printf("Cell is flagged. Unflag it first to open.\n");
                continue;
            }
            if (mine_map[r][c] == 'M') {
                board[r][c] = 'M';
                print_board();
                printf("You stepped on a mine! Game Over!\n");
                break;
            }
            reveal_cell(r, c);
            if (check_win()) {
                print_board();
                printf("Congratulations! You've cleared the minefield!\n");
                break;
            }
        } else { // Assume it's a coordinate input
            r = atoi(cmd);
            scanf("%d", &c);
            if (board[r][c] == 'F') {
                printf("Cell is flagged. Unflag it first to open.\n");
                continue;
            }
            if (mine_map[r][c] == 'M') {
                board[r][c] = 'M';
                print_board();
                printf("You stepped on a mine! Game Over!\n");
                break;
            }
            reveal_cell(r, c);
            if (check_win()) {
                print_board();
                printf("Congratulations! You've cleared the minefield!\n");
                break;
            }
        }
    }

    return 0;
}
```
