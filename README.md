# airplane : 지형 만들기 및 작품 과제 코드입니다

let cols, rows;
let scl = 20; // 지형의 스케일
let w = 2000;
let h = 1600;

let flying = 0;

let terrain = [];

function setup() {
  createCanvas(600, 600, WEBGL);
  cols = w / scl;
  rows = h/ scl;

  for (let x = 0; x < cols; x++) {
    terrain[x] = [];
    for (let y = 0; y < rows; y++) {
      terrain[x][y] = 0; // 초기 지형 높이
    }
  }
}

function draw() {
  flying -= 0.1;
  let yoff = flying;
  for (let y = 0; y < rows; y++) {
    let xoff = 0;
    for (let x = 0; x < cols; x++) {
      terrain[x][y] = map(noise(xoff, yoff), 0, 1, -100, 100);
      xoff += 0.2;
    }
    yoff += 0.2;
  }

  background(135, 206, 235); // 하늘색 배경
  translate(0, 50);
  rotateX(PI / 3);
  fill(144, 238, 144, 200); // 지형 색상 및 투명도
  translate(-w / 2, -h / 2);
  for (let y = 0; y < rows - 1; y++) {
    beginShape(TRIANGLE_STRIP);
    for (let x = 0; x < cols; x++) {
      vertex(x * scl, y * scl, terrain[x][y]);
      vertex(x * scl, (y + 1) * scl, terrain[x][y + 1]);
    }
    endShape();
  }

  // 도형 추가 예제 (간단한 비행기)
  translate(w / 2, h / 3, 100);
  rotateX(-PI / 6);
  fill(255, 0, 0); // 도형 색상
  box(100, 20, 20); // 비행기 몸체
  translate(0, 0, -10);
  box(60, 80, 10); // 비행기 날개

  // 조명 추가
  ambientLight(255, 255, 255); // 전반적인 조명
  pointLight(250, 250, 250, 0, -50, 50); // 특정 위치에서 오는 조명
}


### 이미지 캡쳐

<img width="591" alt="스크린샷 2024-04-07 오후 3 27 42" src="https://github.com/xhitee/graphics/assets/138651672/02fc55a5-075b-4b59-adf2-e68f83477236">

# Snake 게임의 변형

### 벽 기능을 추가하여 플레이어가 벽에 닿으면 game over되도록 수정하였습니다

//Declare Global Variables
let s;
let scl = 20;
let food;
let playfield = 600;
let walls = []; // 벽 배열

// p5js Setup function - required
function setup() {
  createCanvas(playfield, 640);
  background(51);
  s = new Snake();
  frameRate(10);
  pickLocation();
  // 벽 생성
  createWalls();
}

// p5js Draw function - required
function draw() {
  background(51);
  scoreboard();
  if (s.eat(food)) {
    pickLocation();
  }
  s.death();
  s.update();
  s.show();
  fill(255, 0, 100);
  rect(food.x, food.y, scl, scl);
  // 벽 그리기
  for (let wall of walls) {
    wall.show();
  }
}

// Pick a location for food to appear
function pickLocation() {
  let cols = floor(playfield / scl);
  let rows = floor(playfield / scl);
  food = createVector(floor(random(cols)), floor(random(rows)));
  food.mult(scl);
  // Check the food isn't appearing inside the tail
  for (let i = 0; i < s.tail.length; i++) {
    let pos = s.tail[i];
    let d = dist(food.x, food.y, pos.x, pos.y);
    if (d < 1) {
      pickLocation();
    }
  }
}

// Create walls around the playfield
function createWalls() {
  // Top wall
  walls.push(new Wall(0, 0, playfield, scl));
  // Bottom wall
  walls.push(new Wall(0, playfield - scl, playfield, scl));
  // Left wall
  walls.push(new Wall(0, scl, scl, playfield - 2 * scl));
  // Right wall
  walls.push(new Wall(playfield - scl, scl, scl, playfield - 2 * scl));
}

// scoreboard
function scoreboard() {
  fill(0);
  rect(0, 600, 600, 40);
  fill(255);
  textFont("Georgia");
  textSize(18);
  text("Score: ", 10, 625);
  text("Highscore: ", 450, 625);
  text(s.score, 70, 625);
  text(s.highscore, 540, 625);
}

// CONTROLS function
function keyPressed() {
  if (keyCode === UP_ARROW) {
    s.dir(0, -1);
  } else if (keyCode === DOWN_ARROW) {
    s.dir(0, 1);
  } else if (keyCode === RIGHT_ARROW) {
    s.dir(1, 0);
  } else if (keyCode === LEFT_ARROW) {
    s.dir(-1, 0);
  }
}

// SNAKE OBJECT
function Snake() {
  this.x = 0;
  this.y = 0;
  this.xspeed = 1;
  this.yspeed = 0;
  this.total = 0;
  this.tail = [];
  this.score = 1;
  this.highscore = 1;

  this.dir = function (x, y) {
    this.xspeed = x;
    this.yspeed = y;
  };

  this.eat = function (pos) {
    let d = dist(this.x, this.y, pos.x, pos.y);
    if (d < 1) {
      this.total++;
      this.score++;
      text(this.score, 70, 625);
      if (this.score > this.highscore) {
        this.highscore = this.score;
      }
      text(this.highscore, 540, 625);
      return true;
    } else {
      return false;
    }
  };

  this.death = function () {
    for (let i = 0; i < this.tail.length; i++) {
      let pos = this.tail[i];
      let d = dist(this.x, this.y, pos.x, pos.y);
      if (d < 1) {
        this.total = 0;
        this.score = 0;
        this.tail = [];
      }
    }
    // 벽과 충돌 검사
    for (let wall of walls) {
      if (wall.collide(this.x, this.y)) {
        this.total = 0;
        this.score = 0;
        this.tail = [];
      }
    }
  };

  this.update = function () {
    if (this.total === this.tail.length) {
      for (let i = 0; i < this.tail.length - 1; i++) {
        this.tail[i] = this.tail[i + 1];
      }
    }
    this.tail[this.total - 1] = createVector(this.x, this.y);
    this.x = this.x + this.xspeed * scl;
    this.y = this.y + this.yspeed * scl;
    this.x = constrain(this.x, 0, playfield - scl);
    this.y = constrain(this.y, 0, playfield - scl);
  };

  this.show = function () {
    fill(255);
    for (let i = 0; i < this.tail.length; i++) {
      rect(this.tail[i].x, this.tail[i].y, scl, scl);
    }
    rect(this.x, this.y, scl, scl);
  };
}

// Wall class
class Wall {
  constructor(x, y, w, h) {
    this.x = x;
    this.y = y;
    this.w = w;
    this.h = h;
  }

  show() {
    fill(255);
    rect(this.x, this.y, this.w, this.h);
  }

  collide(x, y) {
    return x >= this.x && x <= this.x + this.w && y >= this.y && y <= this.y + this.h;
  }
}


# ml5 개인 프로젝트 :  poseNet을 이용한 자세 인식

