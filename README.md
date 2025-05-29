# Quiz-2-DAA
EF234405 Design &amp; Analysis of Algorithms 

| Name           | NRP        |
| ---            | ---        |
| Kinar Naila Fauziah | 5025231001|
| Azkiya Rusyda Zahra |  5025231072 |
| Safa Nadhira  Azzahra | 5025231086  |


# Explanation
```py
import collections
import pygame

class GridWorld:
    def __init__(self, grid, start, goal):
        self.grid = grid
        self.start = start 
        self.goal = goal     

    def is_valid_move(self, position): 
        x, y = position
        return 0 <= x < len(self.grid) and 0 <= y < len(self.grid[0]) and self.grid[x][y] != '#'

    def get_neighbors(self, position):
        x, y = position
        directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]
        neighbors = []
        for dx, dy in directions:
            new_position = (x + dx, y + dy)
            if self.is_valid_move(new_position):
                neighbors.append(new_position)
        return neighbors

    def get_points(self, position):
        x, y = position
        return int(self.grid[x][y]) if self.grid[x][y].isdigit() else 0
```
A grid-based navigation environment is represented by the GridWorld class.  A grid, a start position, and a goal position are its initializations.  The is_valid_move method determines whether a position is not an obstacle (#) and is within boundaries.  From a given position, the get_neighbors method determines valid nearby positions (up, down, left, and right).  If a cell includes a digit, the get_points method gives its numeric value; if not, it returns 0.


```py
def dfs_find_all_paths(grid_world):
    if not grid_world.start or not grid_world.goal:
        return [] 

    stack = collections.deque([(grid_world.start, [grid_world.start])])
    all_paths_found = []
    while stack:
        current_pos, current_path = stack.pop()
        if current_pos == grid_world.goal:
            all_paths_found.append(list(current_path))
            continue
        for neighbor in reversed(grid_world.get_neighbors(current_pos)):
            if neighbor not in current_path:
                new_path = list(current_path)
                new_path.append(neighbor)
                stack.append((neighbor, new_path))
    return all_paths_found
```
This DFS function, `dfs_find_all_paths`, finds every path in a GridWorld from the beginning to the end.  It explores the path using a stack, adding legitimate neighbors without going back to nodes.  The path is saved if the objective is accomplished.


```py
CELL_SIZE = 45  
INFO_PANEL_HEIGHT = 150 
GRID_LINE_WIDTH = 1

COLOR_BACKGROUND = (30, 30, 30)
COLOR_EMPTY = (220, 220, 220)
COLOR_WALL = (80, 80, 80)
COLOR_START_SELECT = (144, 238, 144) 
COLOR_GOAL_SELECT = (255, 182, 193)   
COLOR_START_FINAL = (0, 200, 0)
COLOR_GOAL_FINAL = (200, 0, 0)
COLOR_PATH = (0, 100, 200)
COLOR_POINT_CELL_BG = (240, 240, 150)
COLOR_TEXT_ON_CELL = (0, 0, 0)
COLOR_TEXT_INFO = (240, 240, 240)
COLOR_TEXT_ERROR = (255, 100, 100)
COLOR_GRID_LINE = (50, 50, 50)

pygame_font_cell = None
pygame_font_info = None
pygame_font_info_small = None
```
these are the pygame UI constants & functions


```py
def draw_grid_with_pygame(screen, current_grid_world, path_to_draw, temp_start, temp_goal, game_phase):
    for r_idx, row in enumerate(current_grid_world.grid):
        for c_idx, cell_content in enumerate(row):
            rect = pygame.Rect(c_idx * CELL_SIZE, r_idx * CELL_SIZE, CELL_SIZE, CELL_SIZE)
            cell_pos = (r_idx, c_idx)
            
            current_color = COLOR_EMPTY
            is_path_cell = path_to_draw and cell_pos in path_to_draw
            
            if cell_content == '#':
                current_color = COLOR_WALL
            elif cell_content.isdigit() and int(cell_content) > 0:
                 current_color = COLOR_POINT_CELL_BG

            if is_path_cell and cell_pos != current_grid_world.start and cell_pos != current_grid_world.goal:
                current_color = COLOR_PATH
            
            if game_phase == "SELECT_START" or game_phase == "SELECT_GOAL":
                if cell_pos == temp_start:
                    current_color = COLOR_START_SELECT
                elif cell_pos == temp_goal: 
                    current_color = COLOR_GOAL_SELECT
            elif current_grid_world.start == cell_pos:
                 current_color = COLOR_START_FINAL
            elif current_grid_world.goal == cell_pos:
                 current_color = COLOR_GOAL_FINAL

            pygame.draw.rect(screen, current_color, rect)
            pygame.draw.rect(screen, COLOR_GRID_LINE, rect, GRID_LINE_WIDTH)

            text_to_draw_on_cell = ""
            if current_grid_world.start == cell_pos:
                text_to_draw_on_cell = "S"
            elif current_grid_world.goal == cell_pos:
                text_to_draw_on_cell = "G"
            elif game_phase == "SELECT_START" and temp_start == cell_pos:
                 text_to_draw_on_cell = "S?"
            elif game_phase == "SELECT_GOAL" and temp_goal == cell_pos:
                 text_to_draw_on_cell = "G?"
            elif cell_content.isdigit() and int(cell_content) > 0:
                text_to_draw_on_cell = cell_content
            
            if text_to_draw_on_cell:
                text_surface = pygame_font_cell.render(text_to_draw_on_cell, True, COLOR_TEXT_ON_CELL)
                text_rect = text_surface.get_rect(center=rect.center)
                screen.blit(text_surface, text_rect)
```
Using Pygame, the `draw_grid_with_pygame` method creates a grid that graphically depicts the `current_grid_world`.  Depending on the game phase (choosing start or goal), it highlights cells differently and iteratively goes over each cell, deciding on its color based on its content.  Each cell is represented by a rectangle, and words such as "S" for start, "G" for goal, or point values are superimposed.  For user interaction, the function makes sure the grid is visually updated with pathways, selections, and labels.


```py
def run_game_with_pygame_ui(base_grid_data):
    pygame.init()
    init_pygame_fonts()

    grid_rows = len(base_grid_data)
    grid_cols = len(base_grid_data[0])
    
    screen_width = grid_cols * CELL_SIZE
    screen_height = grid_rows * CELL_SIZE + INFO_PANEL_HEIGHT
    screen = pygame.display.set_mode((screen_width, screen_height))
    pygame.display.set_caption("DFS Shortest Path Game - Manual Input")

    info_panel_rect = pygame.Rect(0, grid_rows * CELL_SIZE, screen_width, INFO_PANEL_HEIGHT)
    
    game_phase = "SELECT_START" 
    grid_world_instance = GridWorld(base_grid_data, None, None) 
    
    temp_start_node = None 
    temp_goal_node = None  
    
    shortest_path_to_display = None
    path_len = 0
    points_on_path = 0
    info_panel_messages = []
    current_error_message = "" 

    running = True
    clock = pygame.time.Clock()

    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_ESCAPE:
                    running = False
                if event.key == pygame.K_r: 
                    game_phase = "SELECT_START"
                    grid_world_instance.start = None
                    grid_world_instance.goal = None
                    temp_start_node = None
                    temp_goal_node = None
                    shortest_path_to_display = None
                    path_len = 0
                    points_on_path = 0
                    current_error_message = "Selections reset. Click to set Start." # More direct feedback

            if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1: 
                mouse_x, mouse_y = event.pos
                # Only process clicks if they are within the grid area
                if mouse_y < grid_rows * CELL_SIZE:
                    grid_col_clicked = mouse_x // CELL_SIZE
                    grid_row_clicked = mouse_y // CELL_SIZE
                    current_error_message = "" 

                    if 0 <= grid_row_clicked < grid_rows and 0 <= grid_col_clicked < grid_cols:
                        clicked_cell_pos = (grid_row_clicked, grid_col_clicked)
                        
                        if base_grid_data[grid_row_clicked][grid_col_clicked] == '#':
                            current_error_message = "Cannot select a wall (#)!"
                        else:
                            if game_phase == "SELECT_START":
                                temp_start_node = clicked_cell_pos
                                grid_world_instance.start = temp_start_node 
                                game_phase = "SELECT_GOAL"
                            elif game_phase == "SELECT_GOAL":
                                if clicked_cell_pos == temp_start_node:
                                    current_error_message = "Goal cannot be the same as Start!"
                                else:
                                    temp_goal_node = clicked_cell_pos
                                    grid_world_instance.goal = temp_goal_node 
                                    
                                    game_phase = "CALCULATING"
                                    info_panel_messages = [("Calculating path...", COLOR_TEXT_INFO)]
                                    # Draw the "Calculating" state once before blocking for DFS
                                    screen.fill(COLOR_BACKGROUND)
                                    draw_grid_with_pygame(screen, grid_world_instance, shortest_path_to_display, temp_start_node, temp_goal_node, game_phase) # Show S and G selected
                                    draw_info_panel(screen, info_panel_messages, info_panel_rect, current_error_message)
                                    pygame.display.flip()
                                    
                                    print("\n⏳ Searching for paths...") 

                                    all_paths = dfs_find_all_paths(grid_world_instance)
                                    
                                    if not all_paths:
                                        print("No paths found.")
                                        shortest_path_to_display = None
                                        path_len = 0
                                        points_on_path = 0
                                        game_phase = "ERROR_NO_PATH"
                                    else:
                                        print(f"Found {len(all_paths)} path(s).")
                                        shortest_path_to_display = min(all_paths, key=len)
                                        min_len_nodes = len(shortest_path_to_display)
                                        path_len = min_len_nodes -1
                                        points_on_path = sum(grid_world_instance.get_points(pos) for pos in shortest_path_to_display)
                                        print(f"Shortest path length: {path_len}, Points: {points_on_path}")
                                        game_phase = "DISPLAY_PATH"
                    else:
                        current_error_message = "Clicked outside the grid!"
                else:
                    current_error_message = "Clicked in info panel. Click on grid to select."


        if game_phase == "SELECT_START":
            info_panel_messages = [
                ("Click on a non-wall grid cell to set the START point.", COLOR_TEXT_INFO),
                ("Press 'R' to reset selections at any time.", COLOR_TEXT_INFO, "small")
            ]
        elif game_phase == "SELECT_GOAL":
            info_panel_messages = [
                ("Click on a non-wall grid cell to set the GOAL point.", COLOR_TEXT_INFO),
                (f"Start is set at: {temp_start_node}", COLOR_TEXT_INFO, "small"),
                ("Press 'R' to reset.", COLOR_TEXT_INFO, "small")
            ]
        elif game_phase == "CALCULATING":
             info_panel_messages = [("⏳ Calculating path...", COLOR_TEXT_INFO, "small")]
        elif game_phase == "DISPLAY_PATH":
            info_panel_messages = [
                (f"Path found! Length: {path_len} moves.", COLOR_TEXT_INFO),
                (f"Points on this path: {points_on_path}.", COLOR_TEXT_INFO),
                ("Press 'R' to select new Start/Goal points.", COLOR_TEXT_INFO, "small")
            ]
        elif game_phase == "ERROR_NO_PATH":
            info_panel_messages = [
                ("No path found between selected points!", COLOR_TEXT_ERROR),
                (f"Start: {grid_world_instance.start}, Goal: {grid_world_instance.goal}", COLOR_TEXT_INFO, "small"),
                ("Press 'R' to try again.", COLOR_TEXT_INFO, "small")
            ]

        screen.fill(COLOR_BACKGROUND)
        draw_grid_with_pygame(screen, grid_world_instance, shortest_path_to_display, temp_start_node, temp_goal_node, game_phase)
        draw_info_panel(screen, info_panel_messages, info_panel_rect, current_error_message)
        pygame.display.flip()
        
        clock.tick(30)

    pygame.quit()
```
This section is the main game logic with a PYGAME user interface. This is a dynamic update to the gaming interface based on the current gaming phase. Depending on the value of game_phase, various messages in the Information Panel are created to guide the user or display relevant information. During the "select_start" phase, the user is instructed to click on a cell other than the wall to determine it as the starting point while being asked to select the target point for the "select_goal" phase. In the Calculation phase, the message indicates that the path is calculated. If a path is found ("display_path"), the information field indicates the length of the path that was collected and the points that were collected, but in the "error_no_path" phase, an error message notifies the user that there are no paths between the selected points.
