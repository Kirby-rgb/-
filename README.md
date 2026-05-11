extends PanelContainer

# --- 1. 變數宣告與路徑引用 ---
var is_dragging = false
var drag_offset = Vector2.ZERO

# 根據場景樹結構自動獲取節點
@onready var title_bar = $VBoxContainer/TitleBar
@onready var body_label = $VBoxContainer/RichTextLabel
@onready var attachment_btn = $VBoxContainer/AttachmentButton

# 解密介面節點引用
@onready var decryption_overlay = $DecryptionOverlay
@onready var password_field = $DecryptionOverlay/PasswordField
@onready var decrypt_btn = $DecryptionOverlay/VBoxContainer/DecryptBtn

func _ready():
	# 初始化視窗大小與位置，防止介面縮小
	custom_minimum_size = Vector2(800, 600)
	set_anchors_and_offsets_preset(Control.PRESET_CENTER)
	decryption_overlay.hide()
	
	# 手動確保訊號連接，防止失效
	if not attachment_btn.pressed.is_connected(_on_pdf_clicked):
		attachment_btn.pressed.connect(_on_pdf_clicked)
	if not decrypt_btn.pressed.is_connected(_on_decrypt_attempt):
		decrypt_btn.pressed.connect(_on_decrypt_attempt)
	
	title_bar.gui_input.connect(_on_title_bar_gui_input)
	
	# 尋找右上角的關閉按鈕並連結警告邏輯
	var close_btn = find_child("CloseBtn")
	if close_btn:
		if not close_btn.pressed.is_connected(_on_close_denied):
			close_btn.pressed.connect(_on_close_denied)

	# 執行初始打字效果
	run_typing_effect(body_label.text)

# --- 2. 核心效果函數 ---

func run_typing_effect(target_text: String):
	body_label.text = target_text
	body_label.visible_characters = 0
	var tween = create_tween()
	# 打字機動畫
	tween.tween_property(body_label, "visible_characters", target_text.length(), 2.5)
	# 打字結束後觸發閃爍
	tween.finished.connect(_start_button_flash)

func _start_button_flash():
	# 建立無限循環的閃爍效果
	var btn_tween = create_tween().set_loops()
	btn_tween.tween_property(attachment_btn, "modulate", Color.RED, 0.6)
	btn_tween.tween_property(attachment_btn, "modulate", Color.WHITE, 0.6)

func _on_close_denied():
	# 視窗震動效果
	var shake = create_tween()
	for i in 6:
		shake.tween_property(self, "position", position + Vector2(randf_range(-10, 10), 0), 0.05)
	
	# 顯示紅色警告文字
	body_label.modulate = Color.RED
	run_typing_effect("[ 嚴重錯誤 ]：偵測到非法終止嘗試。系統遭『影行者』接管。請立即分析附件。")

# --- 3. 解密小遊戲邏輯 ---

func _on_pdf_clicked():
	decryption_overlay.show()
	# 增加安全檢查，防止 null 報錯
	await get_tree().process_frame
	if password_field:
		password_field.grab_focus()

func _on_decrypt_attempt():
	if password_field.text == "5000":
		_start_unlock_sequence()
	else:
		password_field.text = ""
		password_field.placeholder_text = "密鑰錯誤..."
		# 錯誤反饋動畫
		var t = create_tween()
		password_field.modulate = Color.RED
		t.tween_property(password_field, "modulate", Color.WHITE, 0.2)

func _start_unlock_sequence():
	# 隱藏輸入介面，準備跑代碼特效
	$DecryptionOverlay/VBoxContainer.hide()
	password_field.hide()
	
	body_label.modulate = Color.GREEN
	body_label.text = ">>> [ 正在繞過系統防火牆... ]\n"
	
	var t = create_tween()
	# 駭客亂碼動效
	for i in 30:
		t.tween_callback(func(): 
			body_label.text += str(char(randi_range(33, 126)))
		)
		t.tween_interval(0.02)
	
	t.finished.connect(func():
		decryption_overlay.hide()
		body_label.text = ">>> [ 解碼完成 ]\n\n這不是退稅單。這是一份... [ 零信任監控清單 ]。\n林教授，你也在名單上。"
	)

# --- 4. 視窗拖拽邏輯 ---
func _on_title_bar_gui_input(event):
	if event is InputEventMouseButton and event.button_index == MOUSE_BUTTON_LEFT:
		is_dragging = event.pressed
		drag_offset = get_global_mouse_position() - global_position
	if event is InputEventMouseMotion and is_dragging:
		global_position = get_global_mouse_position() - drag_offset
