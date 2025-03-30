# g- Xử lý sự kiện (Input)
for event in pygame.event.get():
    if event.type == pygame.QUIT:
        running = False

    # Xử lý nhấn phím
    if event.type == pygame.KEYDOWN:
        if event.key == pygame.K_LEFT:
            player_x_change = -player_speed
        if event.key == pygame.K_RIGHT:
            player_x_change = player_speed
        if event.key == pygame.K_SPACE:
            # Chỉ bắn khi đạn trước đã bay xa hoặc chưa bắn
            # (Có thể bỏ điều kiện này nếu muốn bắn liên tục)
            # if bullet_state == "ready":
            fire_bullet(player_x, player_y)

    # Xử lý thả phím
    if event.type == pygame.KEYUP:
        if event.key == pygame.K_LEFT or event.key == pygame.K_RIGHT:
            player_x_change = 0

# --- Cập nhật trạng thái Game ---

if not is_game_over:
    # Cập nhật vị trí người chơi
    player_x += player_x_change
    # Giới hạn di chuyển trong màn hình
    if player_x <= 0:
        player_x = 0
    elif player_x >= screen_width - player_width:
        player_x = screen_width - player_width

    # --- Quản lý kẻ địch ---
    enemy_timer += 1
    if enemy_timer >= enemy_spawn_rate:
        enemy_timer = 0
        enemy_x = random.randint(0, screen_width - enemy_width)
        enemy_y = -enemy_height # Xuất hiện từ mép trên
        enemies.append([enemy_x, enemy_y])

    # Di chuyển kẻ địch và kiểm tra va chạm với người chơi
    enemies_to_remove = []
    for i in range(len(enemies)):
        enemies[i][1] += enemy_speed # Di chuyển xuống

        # Kiểm tra va chạm giữa gà và người chơi
        collision_player = is_collision(player_x, player_y, player_width, player_height,
                                        enemies[i][0], enemies[i][1], enemy_width, enemy_height)
        if collision_player:
            is_game_over = True
            # Không cần break, để vẽ hết gà hiện tại rồi mới hiện Game Over
            # break

        # Xóa gà nếu ra khỏi màn hình
        if enemies[i][1] > screen_height:
            enemies_to_remove.append(i)

    # Xóa gà đã xử lý (đi ra khỏi màn hình) - duyệt ngược để tránh lỗi index
    for index in sorted(enemies_to_remove, reverse=True):
        del enemies[index]


    # --- Quản lý đạn ---
    bullets_to_remove = []
    enemies_hit_indices = set() # Dùng set để tránh xóa cùng 1 gà nhiều lần trong 1 frame

    for i in range(len(bullets)):
        bullets[i][1] -= bullet_speed # Di chuyển đạn lên

        # Kiểm tra va chạm giữa đạn và gà
        for j in range(len(enemies)):
             # Chỉ kiểm tra nếu gà chưa bị đánh dấu để xóa trong frame này
            if j not in enemies_hit_indices:
                collision_bullet_enemy = is_collision(bullets[i][0], bullets[i][1], bullet_width, bullet_height,
                                                    enemies[j][0], enemies[j][1], enemy_width, enemy_height)
                if collision_bullet_enemy:
                    score_value += 1
                    bullets_to_remove.append(i) # Đánh dấu đạn cần xóa
                    enemies_hit_indices.add(j) # Đánh dấu gà cần xóa
                    # Có thể thêm hiệu ứng nổ ở đây
                    break # Một viên đạn chỉ trúng một con gà

        # Xóa đạn nếu bay ra khỏi màn hình
        if bullets[i][1] < -bullet_height: # Kiểm tra vị trí y của đạn
             if i not in bullets_to_remove: # Tránh thêm trùng lặp
                bullets_to_remove.append(i)


    # Xóa đạn đã xử lý (va chạm hoặc ra khỏi màn hình) - duyệt ngược
    for index in sorted(list(set(bullets_to_remove)), reverse=True): # Dùng set để loại bỏ trùng lặp index đạn
        if index < len(bullets): # Kiểm tra index hợp lệ trước khi xóa
           del bullets[index]


    # Xóa gà đã bị bắn trúng - duyệt ngược
    for index in sorted(list(enemies_hit_indices), reverse=True):
         if index < len(enemies): # Kiểm tra index hợp lệ
            del enemies[index]


# --- Vẽ lên màn hình (Render) ---
screen.fill((0, 0, 0)) # Màu nền đen (hoặc vẽ ảnh nền)
# if 'background' in locals():
#     screen.blit(background, (0, 0))

# Vẽ người chơi
draw_player(player_x, player_y)

# Vẽ tất cả kẻ địch
for enemy in enemies:
    draw_enemy(enemy[0], enemy[1])

# Vẽ tất cả đạn
for bullet in bullets:
    draw_bullet(bullet[0], bullet[1])

# Vẽ điểm số
show_score(text_x, text_y)

# Hiển thị Game Over nếu cần
if is_game_over:
    game_over_text()

# 4. Cập nhật màn hình hiển thị
pygame.display.flip() # Hoặc pygame.display.update()

# 5. Giới hạn FPS (Frames Per Second)
clock.tick(60) # Giới hạn game chạy ở 60 FPS
