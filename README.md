#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <string.h>

#define MAX_LEVEL 30
#define MAX_SIZE 50

typedef enum { NORTH = 0, EAST = 1, SOUTH = 2, WEST = 3 } Dir;//create our own data type 

typedef struct {
    int r, c;// r-row position & c-column position
    Dir d;
} Robot;

// Directions
int dr[4] = {-1, 0, 1, 0};
int dc[4] = {0, 1, 0, -1};

char grid[MAX_SIZE][MAX_SIZE];

// Print controls
void print_instructions() {
    printf("=============================================\n");
    printf("             RUN ROBOT GRID GAME\n");
    printf("=============================================\n");
    printf("GOAL: Reach the 'G' position in the grid!\n\n");
    printf("Commands you can use:\n");
    printf("  MOVE  - Move one step forward\n");
    printf("  LEFT  - Rotate 90 degrees left\n");
    printf("  RIGHT - Rotate 90 degrees right\n");
    printf("  EXIT  - Quit the game\n");
    printf("  NEXT  - Go to next level\n");
    printf("=============================================\n\n");
}

// Print Grid
void print_grid(Robot *rb, int rows, int cols) {
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (r == rb->r && c == rb->c) {
                if (rb->d == NORTH) printf("^");
                else if (rb->d == EAST) printf(">");
                else if (rb->d == SOUTH) printf("v");
                else if (rb->d == WEST) printf("<");
            } else {
                printf("%c", grid[r][c]);
            }
        }
        printf("\n");
    }
}

// Random grid generator
void generate_level(int level, Robot *rb, int *rows, int *cols) {
    *rows = 7 + level;      // grows each level
    *cols = 10 + level;     // grows each level

    if (*rows > MAX_SIZE) *rows = MAX_SIZE;
    if (*cols > MAX_SIZE) *cols = MAX_SIZE;

    // Fill with empty space
    for (int r = 0; r < *rows; r++)
        for (int c = 0; c < *cols; c++)
            grid[r][c] = '.';

    // Add border walls
    for (int i = 0; i < *rows; i++) {
        grid[i][0] = '#';
        grid[i][*cols - 1] = '#';
    }
    for (int j = 0; j < *cols; j++) {
        grid[0][j] = '#';
        grid[*rows - 1][j] = '#';
    }

    // Random walls
    int walls = (*rows * *cols) / 8;  
    for (int i = 0; i < walls; i++) {
        int r = rand() % *rows;
        int c = rand() % *cols;
        if (grid[r][c] == '.' && r > 1 && c > 1)
            grid[r][c] = '#';
    }

    // Random goal position
    int gr, gc;
    do {
        gr = rand() % *rows;
        gc = rand() % *cols;
    } while (grid[gr][gc] != '.');
    grid[gr][gc] = 'G';

    // Random robot start
    do {
        rb->r = rand() % *rows;
        rb->c = rand() % *cols;
    } while (grid[rb->r][rb->c] != '.');

    rb->d = EAST;
}

// Move robot
void move_forward(Robot *rb) {
    int nr = rb->r + dr[rb->d];
    int nc = rb->c + dc[rb->d];

    if (grid[nr][nc] == '#') {
        printf("You hit a wall! Can't move forward.\n");
        return;
    }

    rb->r = nr;
    rb->c = nc;
}

// Main game
int main() {
    srand(time(NULL));

    int level = 1;
    char command[20];
    Robot robot;
    int rows, cols;

    print_instructions();

START_LEVEL:
    if (level > MAX_LEVEL) {
        printf("\nYou completed ALL 30 LEVELS! Congratulations!\n");
        return 0;
    }

    printf("\n====== STARTING LEVEL %d ======\n", level);

    generate_level(level, &robot, &rows, &cols);
    print_grid(&robot, rows, cols);

    while (1) {
        printf("\nEnter command: ");
        scanf("%s", command);

        // uppercase command
        for (int i = 0; command[i]; i++)
            if (command[i] >= 'a' && command[i] <= 'z')
                command[i] -= 32;

        if (strcmp(command, "MOVE") == 0) {
            move_forward(&robot);
        }
        else if (strcmp(command, "LEFT") == 0) {
            robot.d = (Dir)((robot.d + 3) % 4);
        }
        else if (strcmp(command, "RIGHT") == 0) {
            robot.d = (Dir)((robot.d + 1) % 4);
        }
        else if (strcmp(command, "EXIT") == 0) {
            printf("Game exited.\n");
            return 0;
        }
        else if (strcmp(command, "NEXT") == 0) {
            level++;
            goto START_LEVEL;
        }
        else {
            printf("Unknown command! Use MOVE, LEFT, RIGHT, EXIT, NEXT.\n");
            continue;
        }

        print_grid(&robot, rows, cols);

        if (grid[robot.r][robot.c] == 'G') {
            printf("\n Congratulations! You reached the goal!\n");
            printf("Type NEXT to go to the next level.\n");
        }
    }

    return 0;
}
