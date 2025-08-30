<?php
// snake.php
// Single-file Snake game: PHP untuk menyimpan high score, HTML/CSS/JS untuk game.

// --- simple highscore storage (file) ---
$scoreFile = __DIR__ . '/snake_highscore.txt';
if (!file_exists($scoreFile)) file_put_contents($scoreFile, "0");
$highscore = intval(file_get_contents($scoreFile));

// Handling AJAX save-highscore
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['save_score'])) {
    $new = intval($_POST['save_score']);
    $current = intval(file_get_contents($scoreFile));
    if ($new > $current) {
        file_put_contents($scoreFile, (string)$new);
        echo json_encode(['status'=>'ok','highscore'=>$new]);
    } else {
        echo json_encode(['status'=>'ok','highscore'=>$current]);
    }
    exit;
}
?>
<!doctype html>
<html lang="id">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Game Ular (Snake) - PHP</title>
<style>
  body { font-family: Arial, Helvetica, sans-serif; background:#111; color:#eee; display:flex; align-items:center; justify-content:center; height:100vh; margin:0; }
  .wrap { text-align:center; }
  canvas { background:#000; display:block; margin:10px auto; border:4px solid #222; image-rendering:pixelated; }
  .info { margin:6px 0; }
  button { padding:8px 12px; border-radius:6px; border:none; background:#28a; color:white; cursor:pointer; }
  button:active{ transform:translateY(1px); }
  .small { font-size:0.9rem; color:#aaa; }
</style>
</head>
<body>
  <div class="wrap">
    <h1>Ular (Snake)</h1>
    <div class="info">Skor: <span id="score">0</span> — Highscore: <span id="highscore"><?= htmlspecialchars($highscore) ?></span></div>
    <canvas id="game" width="400" height="400"></canvas>
    <div>
      <button id="startBtn">Mulai / Restart</button>
    </div>
    <p class="small">Kontrol: Panah / WASD. Makan buah untuk tambah panjang. Jangan menabrak dinding atau badan sendiri.</p>
  </div>

<script>
(() => {
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');
  const startBtn = document.getElementById('startBtn');
  const scoreEl = document.getElementById('score');
  const highEl = document.getElementById('highscore');

  const TILE = 20; // ukuran sel
  const COLS = canvas.width / TILE;
  const ROWS = canvas.height / TILE;

  let snake, dir, apple, score, running, speed;

  function reset
