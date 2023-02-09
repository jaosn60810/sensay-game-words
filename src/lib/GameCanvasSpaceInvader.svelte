<script>
  // @ts-nocheck

  import kaboom from 'kaboom';
  import { onMount } from 'svelte';
  import playerSpriteUrl from '../assets/sprites/player-sprite.png';
  import alienSpriteUrl from '../assets/sprites/alien-sprite.png';

  let player;
  let myGameDivRef;

  export function spawnKaboom() {
    add([
      rect(2, 6),
      pos(player.pos),
      origin('center'),
      color(255, 255, 255),
      area(),
      cleanup(),
      'missile',
      'bullet',
      {
        direction: -1,
      },
    ]);
  }

  onMount(() => {
    // content here
    //IMPORTANT: Make sure to use Kaboom version 0.5.0 for this game by adding the correct script tag in the HTML file.

    kaboom({
      // width: document.body.clientWidth > 640 ? 640 : document.body.clientWidth,
      // height:
      //   document.body.clientWidth * 0.75 > 480
      //     ? 480
      //     : document.body.clientWidth * 0.75,
      width: 800,
      height: 600,
      global: true,
      stretch: false,

      scale: 1,
      debug: true,
      background: [0, 0, 1, 1],
      root: document.querySelector(`#mygame`),
    });

    // CONSTANTS
    const ALIEN_ROWS = 5;
    const ALIEN_COLS = 6;
    const ALIEN_SPEED = 15;
    const ALIEN_STEPS = 322;
    const ALIEN_ROWS_MOVE = 7;
    const ALIEN_SHOOT_COOLDOWN_TIME = 5;

    const BLOCK_HEIGHT = 40;
    const BLOCK_WIDTH = 32;

    const OFFSET_X = 208;
    const OFFSET_Y = 100;

    const PLAYER_MOVE_SPEED = 500;
    const GUN_COOLDOWN_TIME = 1;
    const BULLET_SPEED = 300;

    const POINTS_PER_ALIEN = 100;

    const SCREEN_EDGE = 100;

    // Game Logic

    loadSpriteAtlas(alienSpriteUrl, {
      alien: {
        x: 0,
        y: 0,
        width: 48,
        height: 12,
        sliceX: 4,
        sliceY: 1,
        anims: {
          fly: { from: 0, to: 1, speed: 4, loop: true },
          explode: { from: 2, to: 3, speed: 8, loop: true },
        },
      },
    });

    loadSpriteAtlas(playerSpriteUrl, {
      player: {
        x: 0,
        y: 0,
        width: 180,
        height: 30,
        sliceX: 3,
        sliceY: 1,
        anims: {
          move: { from: 0, to: 0, speed: 4, loop: false },
          explode: { from: 1, to: 2, speed: 8, loop: true },
        },
      },
    });

    // Game Scenes
    scene('game', () => {
      // Player
      player = add([
        sprite('player'),
        scale(1),
        // @ts-ignore
        origin('center'),
        pos(50, 550),
        area(),
        {
          score: 0,
          lives: 3,
        },
        'player',
      ]);

      player.play('move');

      let pause = false;
      onKeyDown('left', () => {
        if (pause) return;
        if (player.pos.x >= SCREEN_EDGE) {
          player.move(-1 * PLAYER_MOVE_SPEED, 0);
        }
      });

      onKeyDown('right', () => {
        if (pause) return;
        if (player.pos.x <= width() - SCREEN_EDGE) {
          player.move(PLAYER_MOVE_SPEED, 0);
        }
      });

      let lastShootTime = time();

      onKeyPress('space', () => {
        if (pause) return;
        if (time() - lastShootTime > GUN_COOLDOWN_TIME) {
          lastShootTime = time();
          spawnBullet(player.pos, -1, 'bullet');
        }
      });

      function spawnBullet(bulletPos, direction, tag) {
        add([
          rect(2, 6),
          pos(bulletPos),
          origin('center'),
          color(255, 255, 255),
          area(),
          cleanup(),
          'missile',
          tag,
          {
            direction,
          },
        ]);
      }

      player.onCollide('alienBullet', (bullet) => {
        if (pause) return;
        destroyAll('bullet');
        player.play('explode');
        updateLives(-1);
        pause = true;
        wait(2, () => {
          if (player.lives == 0) {
            go('gameOver', player.score);
          } else {
            player.moveTo(50, 550);
            player.play('move');
            pause = false;
          }
        });
      });

      // Missiles
      onUpdate('missile', (missile) => {
        if (pause) return;
        missile.move(0, BULLET_SPEED * missile.direction);
      });

      onCollide('bullet', 'alien', (bullet, alien) => {
        destroy(bullet);
        alien.play('explode');
        alien.use(lifespan(0.5, { fade: 0.1 }));
        alienMap[alien.row][alien.col] = null; // Mark the alien as dead
        updateScore(POINTS_PER_ALIEN);
      });

      // Aliens

      let alienDirection = 1;
      let alienMoveCounter = 0;
      let alienRowsMoved = 0;

      let alienMap = [];
      function spawnAliens() {
        for (let row = 0; row < ALIEN_ROWS; row++) {
          alienMap[row] = [];
          for (let col = 0; col < ALIEN_COLS; col++) {
            const x = col * BLOCK_WIDTH * 2 + OFFSET_X;
            const y = row * BLOCK_HEIGHT + OFFSET_Y;
            const alien = add([
              pos(x, y),
              sprite('alien'),
              area(),
              scale(4),
              origin('center'),
              'alien',
              {
                row: row,
                col: col,
              },
            ]);
            alien.play('fly');
            alienMap[row][col] = alien;
          }
        }
      }
      spawnAliens();

      onUpdate(() => {
        if (pause) return;

        every('alien', (alien) => {
          alien.move(alienDirection * ALIEN_SPEED, 0);
        });

        alienMoveCounter++;

        if (alienMoveCounter > ALIEN_STEPS) {
          alienDirection = alienDirection * -1;
          alienMoveCounter = 0;
          // moveAliensDown();
        }

        // if (alienRowsMoved > ALIEN_ROWS_MOVE) {
        //   pause = true;
        //   player.play('explode');
        //   wait(2, () => {
        //     go('gameOver', player.score);
        //   });
        // }
      });

      // function moveAliensDown() {
      //   alienRowsMoved++;
      //   every('alien', (alien) => {
      //     alien.moveBy(0, BLOCK_HEIGHT);
      //   });
      // }

      // Find a random alien to make shoot
      loop(ALIEN_SHOOT_COOLDOWN_TIME, () => {
        if (pause) return;
        // Randomly choose a column, then walk up from the
        // bottom row until an alien that is still alive is found

        let row, col;
        col = randi(0, ALIEN_COLS);
        let shooter = null;

        // Look for the first alien in the column that is still alive
        for (row = ALIEN_ROWS - 1; row >= 0; row--) {
          shooter = alienMap[row][col];
          if (shooter != null) {
            break;
          }
        }
        if (shooter != null) {
          spawnBullet(shooter.pos, 1, 'alienBullet');
        }
      });

      // Score
      add([
        text('SCORE:', { size: 20, font: 'sink' }),
        pos(100, 40),
        origin('center'),
        layer('ui'),
      ]);

      const scoreText = add([
        text('000000', { size: 20, font: 'sink' }),
        pos(200, 40),
        origin('center'),
        layer('ui'),
      ]);

      function updateScore(points) {
        player.score += points;
        scoreText.text = player.score.toString().padStart(6, '0');
      }

      // Lives
      add([
        text('LIVES:', { size: 20, font: 'sink' }),
        pos(650, 40),
        origin('center'),
        layer('ui'),
      ]);

      const livesText = add([
        text('3', { size: 20, font: 'sink' }),
        pos(700, 40),
        origin('center'),
        layer('ui'),
      ]);

      function updateLives(life) {
        player.lives += life;
        livesText.text = player.lives.toString();
      }
    });

    scene('gameOver', (score) => {
      add([
        text('GAME OVER', { size: 40, font: 'sink' }),
        pos(width() / 2, height() / 2),
        origin('center'),
        layer('ui'),
      ]);

      add([
        text('SCORE: ' + score, { size: 20, font: 'sink' }),
        pos(width() / 2, height() / 2 + 50),
        origin('center'),
        layer('ui'),
      ]);

      onKeyPress('space', () => {
        go('game');
      });
    });

    go('game');

    document.querySelector('canvas').focus();
  });
</script>
