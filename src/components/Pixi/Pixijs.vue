<script setup>
import { ref, reactive, nextTick, onMounted } from "vue";

const pixiRoot = ref(null);
onMounted(() => {
  console.log("pixi root: ", pixiRoot.value);

  // Add the assets to load
  // PIXI.Assets.add("daxiong", "/public/daxion.png");
  // PIXI.Assets.add("duolaameng", "/public/duolaameng.png");
  // PIXI.Assets.add("duolamei", "/public/duolamei.png");
  // PIXI.Assets.add("jingxiang", "/public/jingxiang.png");
  // PIXI.Assets.add("falldown", "/public/falldown.png");

  let rectangle = new PIXI.Graphics();
  // rectangle.lineStyle(4, 0xFF3300, 1);
  rectangle.beginFill(0x66CCFF);
  rectangle.drawRect(0, 0, window.innerWidth - 20, 500);
  rectangle.endFill();
  rectangle.x = 10;
  rectangle.y = 10;


  let line = new PIXI.Graphics();
  line.lineStyle(4, 0xFFFFFF, 1);
  line.moveTo(200, 0);
  line.lineTo(200, 200);
  line.x = 0;
  line.y = 0;


  let line2 = new PIXI.Graphics();
  line2.lineStyle(4, 0xFFFFFF, 1);
  line2.moveTo(200, 600);
  line2.lineTo(200, 300);
  line2.x = 0;
  line2.y = 0;

  
  let line3 = new PIXI.Graphics();
  line3.lineStyle(4, 0xFFFFFF, 1);
  line3.moveTo(200, 200);
  line3.lineTo(window.innerWidth - 60, 200);
  line3.x = 0;
  line3.y = 0;

  
  let line4 = new PIXI.Graphics();
  line4.lineStyle(4, 0xFFFFFF, 1);
  line4.moveTo(200, 300);
  line4.lineTo(window.innerWidth - 60, 300);
  line4.x = 0;
  line4.y = 0;

  //创建一个 Pixi应用 需要的一些参数
  let option = {
    width: 400,
    height: 300,
    transparent: true,
  };
  //创建一个 Pixi应用
  let app = new PIXI.Application(option);
  //获取舞台
  let stage = app.stage;
  app.stage.addChild(rectangle);
  app.stage.addChild(line);
  app.stage.addChild(line2);
  app.stage.addChild(line3);
  app.stage.addChild(line4);

  //获取渲染器
  let renderer = app.renderer;
  renderer.view.style.position = "absolute";
  renderer.view.style.display = "block";
  renderer.autoResize = true;
  renderer.resize(window.innerWidth, window.innerHeight);
  //把 Pixi 创建的 canvas 添加到页面上
  pixiRoot.value.appendChild(renderer.view);

  let su = new SpriteUtilities(PIXI);

  //加载图像，加载完成后执行setup函数
  PIXI.loader.add("/public/daxiong.png").load(setup);

  //创建的精灵，会在多个函数中用到
  let Iori;

  function setup() {
    //创建纹理数组
    let frames = su.filmstrip("/public/daxiong.png", 25, 38);
    //使用sprite函数创建精灵
    Iori = su.sprite(frames);
    Iori.vx = 0;
    Iori.vy = 0;
    //设置精灵的位置并将其添加到舞台上
    Iori.position.set(32, 32);
    //添加精灵到舞台上
    stage.addChild(Iori);
    //设置精灵fps来改变动画效果的速度（默认值为12）
    Iori.fps = 12;
    //定义精灵的状态
    Iori.states = {
      down: 0,
      left: 6,
      right: 10,
      up: 12,
      sit: 8,
      liedown: 10,
      falldown: 10,
      walkDown: [0, 3],
      walkLeft: [5, 7],
      walkRight: [9, 11],
      walkUp: [12, 15],
    };
    //捕获键盘箭头键
    let left = keyboard(37),
      up = keyboard(38),
      right = keyboard(39),
      down = keyboard(40);

    //左箭头键 按下
    left.press = () => {
      //播放精灵的 walkLeft 动画序列并设置精灵的速度
      Iori.playAnimation(Iori.states.walkLeft);
      Iori.vx = -1;
      Iori.vy = 0;
    };
    //左箭头键 释放
    left.release = () => {
      //如果左箭头已被释放，右箭头未按下，并且精灵没有垂直移动，
      //则将 vx 设置为0来停止精灵移动，然后显示精灵的静态状态 left
      if (!right.isDown && Iori.vy === 0) {
        Iori.vx = 0;
        Iori.show(Iori.states.left);
      }
    };
    //其余的箭头键遵循相同的格式
    //Up
    up.press = () => {
      Iori.playAnimation(Iori.states.walkUp);
      Iori.vy = -1;
      Iori.vx = 0;
    };
    up.release = () => {
      if (!down.isDown && Iori.vx === 0) {
        Iori.vy = 0;
        Iori.show(Iori.states.up);
      }
    };
    //Right
    right.press = () => {
      Iori.playAnimation(Iori.states.walkRight);
      Iori.vx = 1;
      Iori.vy = 0;
    };
    right.release = () => {
      if (!left.isDown && Iori.vy === 0) {
        Iori.vx = 0;
        Iori.show(Iori.states.right);
      }
    };
    //Down
    down.press = () => {
      Iori.playAnimation(Iori.states.walkDown);
      Iori.vy = 1;
      Iori.vx = 0;
    };
    down.release = () => {
      if (!up.isDown && Iori.vx === 0) {
        Iori.vy = 0;
        Iori.show(Iori.states.down);
      }
    };

    //开始游戏循环
    gameLoop();
  }

  // 将游戏的当前状态设置为play：
  let state = play;
  function gameLoop() {
    // 循环调用gameLoop
    requestAnimationFrame(gameLoop);
    // 更新当前的游戏状态
    state();
    // 渲染舞台
    renderer.render(stage);
  }

  //使精灵移动的函数
  function play() {
    Iori.x += Iori.vx;
    Iori.y += Iori.vy;
  }

  function keyboard(keyCode) {
    let key = {};
    key.code = keyCode;
    key.isDown = false;
    key.isUp = true;
    key.press = undefined;
    key.release = undefined;
    // 按下按键时的处理程序
    key.downHandler = (event) => {
      if (event.keyCode === key.code) {
        if (key.isUp && key.press) key.press();
        key.isDown = true;
        key.isUp = false;
      }
      event.preventDefault();
    };
    // 按键被松开时的处理程序
    key.upHandler = (event) => {
      if (event.keyCode === key.code) {
        if (key.isDown && key.release) key.release();
        key.isDown = false;
        key.isUp = true;
      }
      event.preventDefault();
    };
    // 添加事件监听器
    window.addEventListener("keydown", key.downHandler.bind(key), false);
    window.addEventListener("keyup", key.upHandler.bind(key), false);
    // 返回key对象
    return key;
  }
});
</script>

<template><div ref="pixiRoot"></div></template>

<style lang="scss"></style>
