/**
 * @fileOverview
 * codedojo.kobe.rpg.enchant.js
 * @version 0.1 (2016/12/12)
 * @requires enchant.js v0.8.0 or later
 * @author marumegane
 *
 * @description
 * RPGの下じきかな？
 * これを下じきにどんどん、変更、拡張していってください。
 *
 */

enchant();

window.onload = function() {

    game = new Game(320, 320);

    // フレーム数/秒
    game.fps = 16;

    // Enterキーをaボタンにする
    game.keybind(13, 'a');

    game.isStart = false; // ゲーム開始フラグ

    game.preload(
        // スタート、エンド 
        'start.png', 'end.png',
        // ボタン 
        'button.png',
        // マップ 
        'map1.png',
        // キャラ 
        'chara0.png', 'chara5.png', 'chara6.png',
        // アバター 
        'avatarBg1.png','avatarBg2.png','avatarBg3.png','avatarPlayer1.gif',
        // モンスター
        'monster/monster1.gif', 'monster/monster2.gif',
        'monster/monster3.gif' ,'monster/monster4.gif',
        'monster/monster5.gif', 'monster/monster6.gif',
        'monster/monster7.gif', 'monster/bigmonster1.gif',
        'monster/bigmonster2.gif',
        // サウンド
        'sounds/lock1.wav','sounds/lock2.wav','sounds/se2.wav','sounds/se6.wav','sounds/shot3.wav','sounds/shot4.wav',
        'sounds/se4.wav','sounds/se8.wav','sounds/bgm08.wav'
    );

    game.onload = function() {

        // ホームのマップを作る
        map = new Map(16, 16);
        map.image = game.assets['map1.png'];
        map.loadData(home.bg1, home.bg2);
        map.collisionData = home.collisionData;

        // 前に表示するマップを作る
        var foregroundMap = new Map(16, 16);
        foregroundMap.image = game.assets['map1.png'];
        foregroundMap.loadData(home.fg);

        // ステージを作って、マップとプレイヤーと友達と森の番人と敵をグループにして、いっしょに動かす
        var stage = new Group();
        game.rootScene.stage = stage;
        stage.addChild(map);

        // プレイヤーを作る
        var player = new Player(96, 152, map);
        game.rootScene.player = player;
        stage.addChild(player);

        // 友達を作る
        var friend1 = new Npc(192, 160, 0, map);
        friend1.noMoving = false; // 動く
        stage.addChild(friend1);

        // 森の番人を作る
        var woodsKeeper = new Npc(50, 56, 1, map);
        woodsKeeper.noMoving = true; // 動かない
        stage.addChild(woodsKeeper);

        // マップを追加
        stage.addChild(foregroundMap);

        // ステージを追加
        game.rootScene.addChild(stage);

        var enemiesCnt = 0; // 敵の数
        var enemies = [];   // 敵を入れる配列
        game.rootScene.enemies = enemies;
        
        // サウンドの設定
        game.bgmSD = game.assets['sounds/bgm08.wav'];
        game.battleSD = game.assets['sounds/se4.wav'];
        game.hitSD = game.assets['sounds/shot4.wav'];
        game.swingSD = game.assets['sounds/shot3.wav'];
        game.monsterLostSD = game.assets['sounds/se4.wav'];
        game.talkSD = game.assets['sounds/se2.wav'];
        game.talkYesSD = game.assets['sounds/lock1.wav'];
        game.talkNoSD = game.assets['sounds/lock2.wav'];
        game.recoverSD = game.assets['sounds/se6.wav'];
        
        // rootSceneの「enterframe」イベントリスナ
        game.rootScene.addEventListener('enterframe', function(e) {
            // BGMのくりかえし
            if (game.isStart === false) {
                game.bgmSD.play();
                // ボリュームの調整（最小:0.0 最大:1.0
                // wavファイルは効く MP3は効かないかも
                game.bgmSD.volume = 0.3;
                // 再生をループさせる
                if(location.protocol === "file:") {
                    game.bgmSD._element.loop = true;
                } else {
                    game.bgmSD.src.loop = true;
                }
                game.isStart = true;  // ゲーム開始フラグをtrueにする
            }

            // HPを書きかえる
            hpLabel.text = 'HP:' + playerStatus.hp;
            // コインを書きかえる
            coinLabel.text = 'COIN:' + playerStatus.coin;

            // マップのスクロール処理
            // ゲームのサイズよりマップのサイズのほうが大きい
            // 半分を過ぎたらマップの位置をずらしていく
            // そのためにXとYの位置を次のやりかたで決める
            // ゲームサイズの半分からプレイヤーの位置を引く、それがゼロより小さかったら、その値、大きかったらゼロにする
            var x = Math.min((game.width  - 16) / 2 - player.x, 0);
            var y = Math.min((game.height - 16) / 2 - player.y, 0);
            // 上で決めたX位置、Y位置が小さすぎるとマップのサイズが足らなくなるので、小さくなり過ぎないように調整する
            // マイナスにした位置からマップを置いて、ゲームサイズと比べる、
            // ゲームサイズより大きかったら、その位置、小さかったらマップとゲームのサイズの差をマップのXとYの位置にする
            x = Math.max(game.width,  x + map.width)  - map.width;
            y = Math.max(game.height, y + map.height) - map.height;
            stage.x = x;
            stage.y = y;
            // プレイヤーを画面の下端に移動すると、ホームシーンから公園シーンに切り替える
            if (player.y > 445) game.pushScene(game.park(player.x, player.y));

            // woodsKeeperに近づいた時の処理(24ピクセル以内)、
            if (woodsKeeper.within(player, 36)) { // withinは当たり判定に距離を指定できる
                player.vx = -player.vx / 2; 
                player.vy = -player.vy / 2; 

                if (game.input.a) { //「a」ボタンを押すと
                    game.talkSD.play();
                    // 森の休憩所シーンを表示する
                    game.pushScene(game.woods());
                }
            }

            // friend1に近づいた時の処理(24ピクセル以内)、
            if (friend1.within(player, 24)) { // withinは当たり判定に距離を指定できる
                // プレイヤーの動きを戻す
                player.vx = -player.vx; 
                player.vy = -player.vy; 
                // 友達1の動きを戻す
                friend1.vx = -friend1.vx; 
                friend1.vy = -friend1.vy; 
            }

            // 敵を作る処理
            if (rand(100) < 5 && enemiesCnt < MAX_ENEMIES ) {
                // 敵を出現させる座標を求める
                var ex = rand(28) * 16 + 16;
                var ey = rand(28) * 16 + 16;
                // 求めた座標のマップ上に当たり判定がなかったら、
                if (!map.hitTest(ex, ey)) {
                    // その位置に敵を作って置く
                    var enemy = new Enemy(ex, ey, 1);
                    stage.addChild(enemy);
                    // 敵の数を1ずつ足す
                    enemiesCnt++; 
                    // 現在のフレーム数をキーに設定する 配列の中で敵の区別がつくようにするため
                    // たとえば、プレイヤーが敵を倒した時、どの敵を消めつさせるかを選ぶ時に役に立つ 
                    enemy.key = game.frame;
                    // 敵を配列に入れる
                    enemies[enemy.key] = enemy;
                }
            }
            // 敵の配列を一つずつ調べて、当たり判定する
            for (var i in enemies) {
                // 現在のシーンがルートシーンの場合で プレイヤーがの敵のどれかに当たったら
                if (game.currentScene === this && enemies[i].within(player, 16)) { // withinは当たり判定に距離を指定できる
                    // 敵を消す
                    enemiesCnt --; // カウントをマイナス1
                    enemies[i].remove(); // 配列から消す
                    game.battleSD.play(); // 音を鳴らす
                    // バトルシーンに突入
                    game.pushScene(game.battle());
                }
            }
        });

        // パッドを作る
        var pad = new Pad();
        pad.x = 0;
        pad.y = 220;
        pad.scale(0.8, 0.8);
        game.rootScene.addChild(pad);

        // a ボタンを作る
        var btn = new TapButton((320/2)-(50/2), 250, 'a');
        game.rootScene.addChild(btn);

        // HPラベルを作る
        var hpLabel = new MutableText(80, 0);
        game.rootScene.addChild(hpLabel);

        // コインのラベルを作る
        var coinLabel = new MutableText(200, 0);
        game.rootScene.addChild(coinLabel);
    }

    // 森の休憩所シーン
    game.woods = function() {
        // シーンを作る
        var scene = new Scene();
        // メッセージを表示する
        scene.addChild(label.message("休けいしますか？<br>1回3コインです。"));

        // 選択を表示する

        // 1つ目の選択を作る
        var select0 = label.select("【はい。】", 320 - 32 * 2);
        // 1つ目の選択のタッチイベントリスナ
        select0.addEventListener('touchstart', function(e) {
            game.talkYesSD.play();
            // コインが足りなければ
            if (playerStatus.coin < 3) {
                // メッセージを表示する
                scene.addChild(label.message("コインが足りません。"));
                // 前のシーンに戻る選択
                var select3 = label.select("【戻る】", 320 - 32 * 2);
                select3.addEventListener('touchstart', function(e) {
                    // 【戻る】タッチで前のシーンに戻る
                    game.popScene();
                });
                scene.addChild(select3);
            } else {
                // コインが足りていれば
                game.recoverSD.play();
                // コインを引く
                playerStatus.coin -= 3;
                // プレイヤーのHPを回復する
                playerStatus.hp = playerStatus.hpMax;
                // 前のシーンに戻る
                game.popScene();
            }
        });
        scene.addChild(select0);

        // 1つ目の選択肢を作る
        var select1 = label.select("【いいえ。】", 320 - 32);
        // 2つ目の選択肢のタッチイベントリスナ
        select1.addEventListener('touchstart', function(e) {
            game.talkNoSD.play();
            // 何もせずに前のシーンに戻る
            game.popScene();
        });
        scene.addChild(select1);
        return scene;
    }

    // 公園シーン
    game.park = function(px, py) {
        // シーンを作る
        var scene = new Scene();
        // マップを作る
        var map = new Map(16, 16);
        map.image = game.assets['map1.png'];
        map.loadData(park.bg1, park.bg2);
        map.collisionData = park.collisionData;

        // ステージを作って、マップとプレイヤーと敵をグループにして、いっしょに動かす
        var stage = new Group();
        scene.stage = stage;
        stage.addChild(map);

        var enemiesCnt = 0; // 敵の数
        var enemies = []; // 敵を入れる配列
        scene.enemies = enemies;

        // プレイヤーを作る
        var player = new Player(px + 8, 16, map);
        scene.player = player;
        stage.addChild(player);

        // シーンに「stage」グループを追加する
        scene.addChild(stage);
        // シーンの「enterframe」イベントリスナ
        scene.addEventListener('enterframe', function(e) {
            // HPラベルを更新する
            hpLabel.text = 'HP:' + playerStatus.hp;
            // コインラベルを更新する
            coinLabel.text = 'COIN:' + playerStatus.coin;

            // マップのスクロール処理
            var x = Math.min((game.width  - 16) / 2 - player.x, 0);
            var y = Math.min((game.height - 16) / 2 - player.y, 0);
            x = Math.max(game.width,  x + map.width)  - map.width;
            y = Math.max(game.height, y + map.height) - map.height;
            stage.x = x;
            stage.y = y;
            // プレイヤーを画面の上端まで移動したら、前のシーンへ
            if (player.y < 1 ) game.popScene();

            // 敵を作る処理
            // 0から100までの間の乱数を出して、それが10より小さい場合で、
            // 敵の数が最大数になっていない場合だけ敵をつくる
            if (rand(100) < 10 && enemiesCnt < MAX_ENEMIES ) {
                // 敵を出現させる座標を求める
                var ex = rand(28) * 16 + 16;
                var ey = rand(28) * 16 + 16;
                // 求めた座標のマップ上に当たり判定がなかったら、
                if (!map.hitTest(ex, ey)) {
                    // 敵を作る
                    var enemy = new Enemy(ex, ey, 0);
                    stage.addChild(enemy);
                    // 敵の数を1ずつカウント
                    enemiesCnt ++;
                    // 現在のフレーム数をキーに設定する 配列の中で敵の区別がつくようにするため
                    // たとえば、プレイヤーが敵を倒した時、どの敵を消滅させるかを選ぶ時に役に立つ 
                    enemy.key = game.frame;
                    // 敵を配列に入れる
                    enemies[enemy.key] = enemy;
                }
            }
            for (var i in enemies) {
                // 現在のシーンが park シーンの場合で プレイヤーがの敵のどれかに当たったら
                if (game.currentScene === this && enemies[i].within(player, 16)) { // withinは当たり判定に距離を指定できる
                    // 敵を消す
                    enemiesCnt --; // カウントをマイナス1
                    enemies[i].remove(); // 配列から消す
                    game.battleSD.play(); // 音を鳴らす
                    // バトルシーンに突入
                    game.hitSD.play();
                    // バトルシーンへ
                    game.pushScene(game.battle());
                }
            }
        });

        // パッドを作る
        var pad = new Pad();
        pad.x = 0;
        pad.y = 220;
        pad.scale(0.8, 0.8);
        scene.addChild(pad);

        // a ボタンを作る
        var btn = new TapButton(120, 250, 'a');
        scene.addChild(btn);

         // HPラベルを作る
        var hpLabel = new MutableText(80, 0);
        scene.addChild(hpLabel);

        // コインラベルを作る
        var coinLabel = new MutableText(200, 0);
        scene.addChild(coinLabel);

       return scene;

    }

    // バトルシーン
    game.battle = function(no) {

        // バトル中フラグを「true」にする
        game.isBattle = true;

        // 攻撃ターン 始めはプレイヤーにしておく
        game.battle.turn = "player";

        // 新しいシーンを作る
        var scene = new Scene();

        // シーンの背景を白色にする 上下の余白を消すため
        scene.backgroundColor="#FFFFFF";
        
        // アバターの背景を作る
        bg = new AvatarBG(1);
        bg.y = 50;
        scene.addChild(bg);

        var monster; // モンスター変数 テーブルの１行ずつを入れる
        // モンスタをランダムに出す設定する
        monster = monstorTable[rand(8)];

        // 武器ゲットフラグを「true」にする
        var isWeaponGet = false;

        // 「monster」変数に設定されたモンスターデータを元にモンスターを作る
        var monsterEnemy = new AvatarMonster(game.assets[monster.image]);
        monsterEnemy.x = 200;
        monsterEnemy.y = 100;
        monsterEnemy.hp = monster.hp * playerStatus.lv; // レベル
        monsterEnemy.attack = monster.attack;           // 攻撃力
        monsterEnemy.exp = monster.exp;                 // 経験値
        monsterEnemy.coin = monster.coin;               // プレイヤーがゲットするコイン
        monsterEnemy.drop = monster.drop;               // ドロップするアイテム
        monsterEnemy.no = no;                           // 種類
        monsterEnemy.vx = -8;                           // 移動量  
        monsterEnemy.lost = false;                      // 消滅フラグ
        monsterEnemy.action = 'appear'                  // アクション
        scene.addChild(monsterEnemy);

        // モンスターの「enterframe」イベントリスナ
        monsterEnemy.addEventListener('enterframe', function() {
            // モンスターの攻撃ターンでなければ何もしない
            if (game.battle.turn !== "monsterEnemy") return;

            // バトル中でなければリターン
            if (game.isBattle === false) return;

            // 「attack」「appear」「disappear」アクションならリターン
            if (this.action !== "attack" && this.action !== "appear" && this.action !== "disappear") {
                // モンスターの移動処理
                this.x += this.vx;
                // バックグラウンドをプレイヤの動きに合わせてスクロールする
                bg.scroll(this.x);
            }

            // プレイヤとの当たり判定

            // プレイヤとモンスターの中心点どうしの距離が「24」ピクセル以下なら
            if (battlePlayer.within(this, 24) && this.action === "walk") { // withinは当たり判定に距離を指定できる
                // 「attack」アクションにする
                this.action = "attack";
                battlePlayer.action = "damage";
                game.hitSD.play();
                this.animFrme = 0;
                this.vx = 8;
                // プレイヤのHPから、攻撃xレベルを引く
                battlePlayer.hp -= this.attack * battlePlayer.lv;
                // プレイヤのHPが「0」以下になったら、プレイヤのHPを「0」にする
                if (battlePlayer.hp < 0) battlePlayer.hp = 0;
                // HP表示ラベルを更新する
                hpLabel.text = 'HP:' + battlePlayer.hp + '/' + battlePlayer.hpMax;
                // プレイヤのHPが「0」以下になったら、ゲームオーバーシーンを表示する
                if (battlePlayer.hp <= 0) game.pushScene(game.lose());
                // 停止攻撃モードなら
            } else if (this.action === "stop" && battlePlayer.action === "stop") {
                // stopになるまでwait
                this.action = "walk";
            } 

            // モンスターのx座標が「200」以上の場合
            if (this.x >= 200) {
                // プレイヤーの攻撃ターンにする
                game.battle.turn = "player";
                // モンスターの向きを元に戻す
                this.vx = -8;
            }
        });

        // 「wp」変数に、現在装備している武器のデータを入れる
        var wp = weapon[playerStatus.weapon];
        // プレイヤーを作る
          // オフラインの場合
          //  var battlePlayer = new AvatarPlayer(game.assets['avatarPlayer1.gif']);
        var battlePlayer = new Avatar("1:2:1:"+ wp.no +":21011:2211");
        scene.addChild(battlePlayer);
        battlePlayer.x = 50;
        battlePlayer.y = 100;
        battlePlayer.scaleX = -1;                  // x方向の向き
        battlePlayer.scaleY = 1;                   // y方向の向き
        battlePlayer.vx = 8;                       // x方向の移動量
        battlePlayer.isMoving = false;             // 移動フラグ
        battlePlayer.lv = playerStatus.lv;         // レベル
        battlePlayer.hp = playerStatus.hp;         // 現在HP
        battlePlayer.hpMax = playerStatus.hpMax;   // 最大HP
        battlePlayer.exp = playerStatus.exp;       // 経験値
        battlePlayer.attack = playerStatus.attack; // 攻撃力
        battlePlayer.coin = playerStatus.coin;     // コイン
        battlePlayer.weapon = playerStatus.weapon; // 武器
        // プレイヤの「enterframe」イベントリスナ
        battlePlayer.addEventListener('enterframe', function() {

            if (game.battle.turn !== "player") return;
            //
            // バトル中でなければリターン
            if (game.isBattle === false) return;

            // モンスターが生存中(画上いるとき)の処理
            if (!monsterEnemy.lost) {

                // プレイヤの攻撃、移動処理

                // 移動中の処理
                if (this.isMoving === true) {
                    // 右方向に「vx」プロパティの値ずつ移動させる
                    this.x += this.vx;
                    // プレイヤを「run」アクション
                    this.action = "run";
                    // バックグラウンドをプレイヤの動きに合わせてスクロールする
                    bg.scroll(this.x);
                    // モンスターと衝突したら停止する
                    if (monsterEnemy.intersect(this)) {
                        this.isMoving = false;
                        // プレイヤを「attack」アクション
                        this.action = "attack";
                        monsterEnemy.action = "attack";
                        // スイングSDを再生する
                        game.swingSD.play();
                        monsterEnemy.animFrme = 0;
                        // モンスターのHPから、プレイヤの攻撃力+武器の攻撃力を引く
                        monsterEnemy.hp -= (this.attack +  wp.attack);
                        // モンスターのHPが「0」以下なら
                        if (monsterEnemy.hp <= 0) {
                            // 消滅フラグを「true」にする
                            monsterEnemy.lost = true;
                            // 所持コインに取得コインを加算する
                            this.coin += monsterEnemy.coin;
                            // 装備中の武器より、モンスターが落とす武器の性能が良ければ、
                            if (this.weapon < monsterEnemy.drop) {
                                // 落とした武器を装備する
                                this.weapon = monsterEnemy.drop;
                                playerStatus.weapon = this.weapon;
                                // 武器ゲットフラグを「true」にする
                                isWeaponGet = true;
                            }
                            // モンスターをシーンから消滅させる
                            monsterEnemy.action = "disappear";
                        }
                    }
                    // 元の位置まで戻ったら停止する
                    if (this.x <= 50) {
                        this.isMoving = false;
                        // プレイヤを「attack」アクション
                        this.action = "stop";
                        // プレイヤを右向きにする
                        this.scaleX = -1;
                        game.battle.turn = "monsterEnemy";
                        monsterEnemy.action = "walk";
                        monsterEnemy.animFrme = 0;
                    }
                } else if (game.input.a && this.x <= 50 && game.battle.turn === "player") {
                    // プレイヤを右向きにする
                    this.scaleX = -1;
                    // プレイヤを「run」アクション
                    this.action = "run";

                    this.isMoving = true;

                    // 右向きにする
                    this.vx = 8; 

                    // 攻撃モードなら
                } else if (this.action === "stop" && monsterEnemy.action === "stop" && this.x > 100) {
                    this.isMoving = true;
                    // プレイヤを左向きにする
                    this.scaleX = 1;
                    // プレイヤを「run」アクション
                    this.action = "run";
                    // 左向きにする
                    this.vx = -8; 
                } 
            } else {
                if (scene.childNodes.indexOf(monsterEnemy) === -1) {
                    game.monsterLostSD.play();
                    // HPを更新
                    playerStatus.hp = this.hp;
                    // 所持コインを更新
                    playerStatus.coin = this.coin;
                    // 勝利シーンを表示する
                    if (game.isBattle) game.pushScene(game.win(isWeaponGet));
                    // バトル終了
                    game.isBattle = false;
                }
            }
        });

        // シーンの「enterframe」イベントリスナ
        scene.addEventListener('enterframe', function() {
            // バトル中でなければ、前のシーン(公園)に戻る
            if (game.isBattle === false) game.popScene();
        });

        // バーチャル「a」ボタンを作る
        var btn = new TapButton((320/2)-(50/2), 250, 'a');
        scene.addChild(btn);

        // 最大HP/現在HP表示ラベルを作る
        var hpLabel = new MutableText(10, 32);
        hpLabel.text = 'HP:' + playerStatus.hp + '/' + playerStatus.hpMax;
        scene.addChild(hpLabel);

        return scene;
    }

    // 勝利シーン
    game.win = function(isWeaponGet) {
        // 新しいシーンを作る
        var scene = new Scene();

        // 表示するメッセージを設定する
        var message = "モンスターを倒した！";

        // メッセージを表示する
        scene.addChild(label.message(message));
        // 武器ドロップがあったら、ゲットした武器の名前をメッセージに入れる
        if (isWeaponGet) {
            var wp = weapon[playerStatus.weapon].name;
            message += "<br>" + wp + "を手に入れた!";
            // メッセージを表示する
            scene.addChild(label.message(message));
        }

        // 【戻る】選択肢を表示する
        var select0 = label.select("【戻る】", 320 - 32);
        select0.addEventListener('touchstart', function(e) {
            game.popScene(); // 【戻る】タッチで前のシーンに戻る
        });
        scene.addChild(select0);

        return scene;
    }

    // ゲームオーバーシーン
    game.lose = function() {
        // 新しいシーンを作る
        var scene = new Scene();
        // メッセージを表示する
        scene.addChild(label.message("モンスターに倒された....."));

        // ゲームオーバー画像のスプライトを作る
        var gameover = new Sprite(189, 97);
        gameover.image = game.assets['end.png'];
        gameover.x = 60;
        gameover.y = 112;
        scene.addChild(gameover);
        game.bgmSD.stop();

        return scene;
    }

    game.start();
}

// ラベル
var label = {
    // メッセージ
    message : function(text) {
        var lbl = new Label(text);
        lbl.font  = "16px monospace";
        lbl.color = "rgb(255,255,255)";
        lbl.backgroundColor = "rgba(0, 0, 0, 1.0)";
        lbl.y     = 320 - 32 * 3;
        lbl.width = 320;
        lbl.height = 32 * 3;
        return lbl;
    },
    // 選択肢を作る関数
    select : function(text, y) {
        var lbl = new Label(text);
        lbl.font  = "16px monospace";
        lbl.color = "rgb(255,200,0)";
        lbl.y     = y;
        lbl.width = 320;
        return lbl;
    }
}

// バーチャルボタンを作るクラス
var TapButton = enchant.Class.create(enchant.Sprite, {
    initialize: function(x, y, mode) {
        enchant.Sprite.call(this, 50, 50);
        this.image = game.assets['button.png'];
        this.x = x;
        this.y = y;
        this.buttonMode = mode; 
        this.scale(0.8, 0.8); // 大きさの倍率 
    }
});

var TapPad = {};
TapPad.prototype = Object.create(enchant.ui.Pad.prototype, {

});
//var tabpad = new TapPad();

// プレイヤーを作るクラス
var Player = enchant.Class.create(enchant.Sprite, {
    initialize: function(x , y, map) {
        enchant.Sprite.call(this, 32, 32);
        this.x = x;
        this.y = y;
        this.hp = playerStatus.hp;   // HP
        // スプライト画像から使う分だけ切り取って、いったんサーフェイスとして書き出す
        var image = new Surface(96, 128);
        image.draw(game.assets['chara5.png'], 0, 0, 96, 128, 0, 0, 96, 128);
        this.image =image;
        this.isMoving = false; // 移動フラグ(移動中なら「true」)
        this.direction = 0;    // 向き
        // 歩行アニメーションの基準フレーム番号を保持するプロパティ
        this.walk = 0;
        // 攻撃アクション中のフレーム数を保持するプロパティ
        this.acount = 0;
        // 「enterframe」イベントリスナ
        this.addEventListener('enterframe', function() {

            // プレイヤーの移動処理

            // 歩行アニメーションのフレーム切り替え
            this.frame = this.direction * 3 + this.walk;
            // 移動中の処理
            if (this.isMoving) {
                // 「vx」「vy」プロパティの分だけ移動する
                this.moveBy(this.vx, this.vy);
                // 歩行アニメーションの基準フレーム番号を取得する
                this.walk = game.frame % 3;
                // 次のマス(16x16が1マス)まで移動しきったら停止する
                // 16の倍数のところにしか止まらないようにする
                // そうすることで、敵から追跡しやすくできるし、
                // マップ上の当たり判定で止める位置を一定にすることができる
                if ((this.vx && (this.x - 8) % 16 === 0) || (this.vy && this.y % 16 === 0)) {
                    this.isMoving = false;
                    this.walk = 0;
                }
            } else {
                // 移動中でないときは、ランダムに移動を決める
                // direction は キャラの種類で vx は キャラの動く方向
                this.vx = this.vy = 0;
                if (game.input.left) {
                    this.direction = 1;
                    this.vx = -4;
                } else if (game.input.right) {
                    this.direction = 2;
                    this.vx = 4;
                } else if (game.input.up) {
                    this.direction = 3;
                    this.vy = -4;
                } else if (game.input.down) {
                    this.direction = 0;
                    this.vy = 4;
                }
                // 移動先が決まったら、(vxもvyもゼロでない値が設定されていれば)
                if (this.vx || this.vy) {
                    // 移動先の座標を求める
                    // vx vy を元に 移動先位置を求め、そこが移動可能かどうか調べる
                    // playerのスプライトのサイズは32なので、右側に接する位置は32、左側に接する位置は 0 中心位置は 16 になる
                    // vx vy が  4 の場合  16 + 16 = 32
                    // vx vy が -4 の場合 -16 + 16 = 0
                    // vx vy が  0 の場合   0 + 16 = 16
                    var x = this.x + (this.vx ? this.vx / Math.abs(this.vx) * 16 : 0) + 16;
                    var y = this.y + (this.vy ? this.vy / Math.abs(this.vy) * 16 : 0) + 16;
                    // その座標が移動可能な場所なら
                    if (0 <= x && x < map.width && 0 <= y && y < map.height && !map.hitTest(x, y)) {
                        // 移動フラグを「true」にする
                        this.isMoving = true;
                    }
                }
            }
        });
    }
});

// 敵の最大数
MAX_ENEMIES = 10; // 敵の最大数

// 敵のスプライトを作るクラス
var Enemy = enchant.Class.create(enchant.Sprite, {
    initialize: function(x, y, kind) {
        enchant.Sprite.call(this, 32, 32);
        this.image = game.assets['chara6.png'];
        this.x = x;
        this.y = y;
        this.isMoving = false; // 移動フラグ(移動中なら「true」)
        this.direction = 0;    // 向き
        // 歩行アニメーションの基準フレーム番号を保持するプロパティ
        this.walk = 0;
        // 敵の種類は 0 か 1
        this.kind = kind;
        // 3 をかけると 0 列目 か 3 列目 かが基準になる そこから向きによって 何行目かを選ぶ
        this.frame = this.kind * 3; 

        // 「enterframe」イベントリスナ
        this.addEventListener('enterframe', function() {

            // 敵の移動処理

            // 3 パターン歩くアニメーションのフレーム切り替え
            this.frame = (this.direction + this.kind) * 3 + this.walk;
            // 移動中の処理
            if (this.isMoving) {
                this.moveBy(this.vx, this.vy);
                this.walk = game.frame % 3;
                if ((this.vx && (this.x-8) % 16 === 0) || (this.vy && this.y % 16 === 0)) {
                    this.isMoving = false;
                    this.walk = 0;
                }
            } else {
                // 移動中でないときは、ランダムに移動を決める
                // direction は キャラの種類で vx は キャラの動く方向
                this.vx = this.vy = 0;
                this.mov = rand(4);
                switch (this.mov) {
                    case 0: 
                        this.direction = 0;
                        this.vy = 4;
                        break;
                    case 1: 
                        this.direction = 2;
                        this.vx = -4;
                        break;
                    case 2: 
                        this.direction = 4;
                        this.vx = 4;
                        break;
                    case 3: 
                        this.direction = 6;
                        this.vy = -4;
                        break;
                }

                // プレイヤーを追いかける処理

                // 自身のxy軸線上に今のシーンのプレイヤーいたら、その方向に移動方向設定する
                var player = game.currentScene.player; 
                if (this.x > player.x && this.y === player.y) {
                    this.direction = 2;
                    this.vx = -4;
                } else if (this.x < player.x && this.y === player.y) {
                    this.direction = 4;
                    this.vx = 4;
                } else if (this.y > player.y && this.x === player.x) {
                    this.direction = 6;
                    this.vy = -4;
                } else if (this.y < player.y && this.x === player.x) {
                    this.direction = 0;
                    this.vy = 4;
                }
                // 移動先が決まったら
                if (this.vx || this.vy) {
                    // 移動先の座標を求める
                    var x = this.x + (this.vx ? this.vx / Math.abs(this.vx) * 16 : 0) + 16;
                    var y = this.y + (this.vy ? this.vy / Math.abs(this.vy) * 16 : 0) + 16;
                    // その座標が移動可能な場所なら
                    if (0 <= x && x < map.width && 0 <= y && y < map.height && !map.hitTest(x, y)) {
                        // 移動フラグを「true」にする
                        this.isMoving = true;
                    }
                }
            }
        });
        return this;
    },
    remove: function() {
        game.currentScene.stage.removeChild(this); // シーンの子供から消す
        delete game.currentScene.enemies[this.key]; // 敵の配列から消す
        delete this; // 敵自身を消す
    }
});

// NPC（Non Player Character）を作るクラス
var Npc = enchant.Class.create(enchant.Sprite, {
    initialize: function(x, y , no , map) {
        enchant.Sprite.call(this, 32, 32);
        this.x = x;
        this.y = y;
        this.kind = no; // NPCの種類
        // NPCの種類ごとにスプライト画像から使う分だけ切り取って、いったんサーフェイスとして書き出す
        var image = new Surface(96, 128);
        switch (this.kind) {
            case 0: image.draw(game.assets['chara0.png'], 0, 0, 96, 128, 0, 0, 96, 128);
                break;
            case 1: image.draw(game.assets['chara0.png'], 96, 0, 96, 128, 0, 0, 96, 128);
                break;
            case 2: image.draw(game.assets['chara0.png'], 192, 0, 96, 128, 0, 0, 96, 128);
                break;
        }
        this.image = image; //サーフィスの画像をスプライトの画像に設定する
        this.isMoving = false; // 移動フラグ(移動中なら「true」)
        this.noMoving = false; // 動くNPCなら「false」、動かないNPCなら「true」
        this.direction = 0;    // 向き
        // 歩くアニメーションのフレーム番号
        this.walk = 0;
        this.frame = 0;
        // 「enterframe」イベントリスナ
        this.addEventListener('enterframe', function() {

            if (this.noMoving) return; // 動かないNPCならリターン

            // NPCの移動処理

            // 3 パターン歩くアニメーションのフレーム切り替え
            this.frame = this.direction * 3 + this.walk;

            // 移動中の処理
            if (this.isMoving) {
                this.moveBy(this.vx, this.vy);
                this.walk = game.frame % 3;
                if ((this.vx && (this.x-8) % 16 === 0) || (this.vy && this.y % 16 === 0)) {
                    this.isMoving = false;
                    this.walk = 0;
                }
            } else {
                // 移動中でないときは、ランダムに移動方向を設定する
                this.vx = this.vy = 0;
                this.mov = rand(4);
                switch (this.mov) {
                    case 0: 
                        this.direction = 0;
                        this.vy = 4;
                        break;
                    case 1: 
                        this.direction = 1;
                        this.vx = -4;
                        break;
                    case 2: 
                        this.direction = 2;
                        this.vx = 4;
                        break;
                    case 3: 
                        this.direction = 3;
                        this.vy = -4;
                        break;
                }
                // 移動先が決まったら
                if (this.vx || this.vy) {
                    // 移動先の座標を求める
                    var x = this.x + (this.vx ? this.vx / Math.abs(this.vx) * 16 : 0) + 16;
                    var y = this.y + (this.vy ? this.vy / Math.abs(this.vy) * 16 : 0) + 16;
                    // その座標が移動可能な場所なら
                    if (0 <= x && x < map.width && 0 <= y && y < map.height && !map.hitTest(x, y)) {
                        // 移動フラグを「true」にする
                        this.isMoving = true;
                    }
                }
            }
        });
    }
});

/**
 * オフライン用のバトル用プレイヤー
 */
AvatarPlayer = enchant.Class.create(enchant.avatar.AvatarCharacter, {
    /**
     * @param {int}code  Avatar code
     * @extends enchant.avatar.AvatarCharacter
     * @constructs
     */
    initialize: function(image) {
        enchant.avatar.AvatarCharacter.call(this, 64, 64);
        this.image = image;
        this.animPattern = { "stop": [ 0],
            "run": [ 1, 2, 3, 2],
            "attack": [ 0, 2, 9, 10, 11, 5, 0, 0, 0, -1],
            "special": [ 0, 6, 5, 13, 14, 15, 0, 0, -1],
            "damage": [ 7, 7, 7, 7, 0, 0, 0, -1],
            "dead": [8],
            "demo": [ 1, 2, 3, 2, 1, 2, 3, 2, 1, 2, 3, 2, 1, 2, 3, 2, 1, 2, 3, 2, 1, 2, 3, 2, 0, 0, 0, 0, 2, 9, 10, 11, 5, 0, 0, 0,
                1, 2, 3, 2, 1, 2, 3, 2, 1, 2, 3, 2, 1, 2, 3, 2, 1, 2, 3, 2, 0, 6, 5, 13, 14, 15, 0, 0]
        }
    }
});

// 定数

// モンスターテーブル
//  image : 画像ファイル名
//  hp    : HP
//  exp   : 経験値
//  attack: 攻撃力
//  coin  : プレイヤーがゲットするコイン
//  drop  : 落とす武器
var monstorTable = {
    0: {image:'monster/monster1.gif', hp:5, exp:10, attack:1, coin:10, drop:3},
    1: {image:'monster/monster2.gif', hp:10, exp:20, attack:2, coin:20, drop:4},
    2: {image:'monster/monster3.gif', hp:15, exp:30, attack:3, coin:30, drop:5},
    3: {image:'monster/monster4.gif', hp:20, exp:40, attack:4, coin:40, drop:6},
    4: {image:'monster/monster5.gif', hp:25, exp:20, attack:5, coin:60, drop:7},
    5: {image:'monster/monster6.gif', hp:30, exp:30, attack:5, coin:60, drop:8},
    6: {image:'monster/monster7.gif', hp:35, exp:50, attack:5, coin:15, drop:9},
    7: {image:'monster/bigmonster1.gif', hp:50, exp:1000, attack:30, coin:1000, drop:13},
    8: {image:'monster/bigmonster2.gif', hp:60, exp:1000, attack:40, coin:1000, drop:14}
}

var playerStatus = {
    lv: 1,         // レベル どこでレベルアップしたらいいか？
    hp: 36,      // 現在HP
    hpMax: 36,   // 最大HP
    exp: 0,        // 経験値 どこで増やすか？
    attack: 1,     // 攻撃力
    coin: 0,       // コイン
    weapon: 0,     // 武器
}

// 武器テーブル
// no    : 番号
// name  : 名前
// attack: 攻撃力
var weapon = {
    0: {no:2002, name:'ブロンズソード', attack:1},
    1: {no:2004, name:'ブラスソード', attack:2},
    2: {no:2005, name:'アイアンソード', attack:3},
    3: {no:2009, name:'スチールソード', attack:4},
    4: {no:2010, name:'ヘヴィソード', attack:5},
    5: {no:2019, name:'ブロードソード', attack:6},
    6: {no:2020, name:'クレイモア', attack:6},
    7: {no:2054, name:'スラッシュレイピア', attack:7},
    8: {no:2055, name:'サーベル', attack:8},
    9: {no:2044, name:'ブレイズソード', attack:9},
    10: {no:2091, name:'ブレイズブレイド', attack:10},
    11: {no:2091, name:'アクアブレイド', attack:11},
    12: {no:2073, name:'バラの宝剣', attack:12},
    13: {no:2098, name:'ドラゴンキラー', attack:13},
    14: {no:2506, name:'王家の剣', attack:14},
    15: {no:2514, name:'ダークブレイド', attack:15},
    16: {no:2597, name:'プロミネンスソード', attack:20},
}

// ホームシーンのマップデータ
var home = {
    'bg1': [
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],
        [20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20]
    ],
    'bg2': [
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,60,61,60,61,60,61,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,76,77,76,77,76,77,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,60,61,-1,-1,60,61,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,76,77,-1,-1,76,77,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,60,61,-1,-1,60,61,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,76,77,-1,-1,76,77,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,60,61,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,76,77,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,60,61,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,76,77,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1]
    ],
    collisionData: [
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,1,1,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,1,1,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
    ],
    fg: [
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,60,61,60,61,60,61,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,60,61,-1,-1,60,61,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,60,61,-1,-1,60,61,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,60,61,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,60,61,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1],
        [-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1]
    ]
} 

// 公園のマップデータ
var park = {
    'bg1': [
        [ 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20],
        [ 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20],
        [ 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20],
        [ 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20],
        [ 20, 20, 20, 20, 20, 83,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20, 83,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20,115,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20, 20,115,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20, 20, 20,115,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100,100],
        [ 20, 20, 20, 20, 20, 20, 20, 20, 20,116,116, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20],
        [ 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20],
        [ 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20, 20],
    ],
    'bg2': [
        [ -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1, -1, -1,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107],
        [ -1, -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1, -1, -1,107, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1, -1, -1, -1,107,107,107, -1, -1,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107,107],
        [ -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
        [ -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
    ],
    collisionData: [
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
        [0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,1,1,1,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
    ]
} 
