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
####1. 프로젝트 목표 및 효과
 실시간 자세 인식이 가능해 운동 및 피트니스에서 동작의 정확성을 평가할 수 있습니다. 재활 운동을 할 때에도 자세를 인식해 올바른 교정이 가능하게 합니다.
 잘못된 자세를 식별하고 교정함으로써 건강 문제를 개선할 수 있습니다. 운동의 효율성을 극대화하고 부상 위험을 줄일 수 있습니다. 온라인으로도 보다 정확한 자세 교정이 가능합니다.

####2. 데이터셋
 사전학습된 모델을 사용하였기에 추가적인 train data는 사용하지 않았습니다. test data는 vision.lab의 구성원인 남자 5명, 여자 2명을 활용했습니다.


####3. 개발 환경
프로젝트 진행에 사용한 pc 세부 사양을 표로 나타내었다.
<img width="887" alt="스크린샷 2024-05-10 오후 7 41 36" src="https://github.com/xhitee/graphics/assets/138651672/521e60b4-3bc9-4b42-af7b-70f90f6ea06a">

vscode에서 소스를 작성했으며 브라우저는 chrome을 사용하였다. 서버는 nodejs를 사용하지 않고 vscode 확장 프로그램인 live server를 사용해 열었다.

####4. 시연 영상

####5. 소감
 이 프로젝트의 진행은 기술과 인간의 삶이 어떻게 상호작용할 수 있는지에 대한 흥미로운 탐색이었습니다. PoseNet 같은 고급 기술을 활용해 일상 생활에서 직면하는 문제, 특히 잘못된 자세로 인한 건강 문제를 해결할 수 있다는 점에서 큰 의의를 찾았습니다. 특히, 사람들이 자신의 자세를 실시간으로 인식하고 개선할 수 있도록 돕는 것은 장기적으로 건강을 개선하는 데 큰 도움이 될 것입니다.
 또한, 이 기술이 운동 효율성을 높이고, 재활 과정을 지원하며, 원격 교육의 질을 향상시킬 수 있는 가능성을 보여주었습니다. 이는 단순히 기술적 성취를 넘어서 사회적, 교육적 측면에서도 긍정적인 영향을 미칠 수 있음을 의미합니다.
 개발 과정에서 마주친 도전과 어려움들은 있었지만, 최종적으로 이 기술이 사람들의 삶의 질을 개선하는 데 기여할 수 있다는 확신을 갖게 되었습니다. 앞으로도 이러한 기술이 더욱 발전하여 더 많은 분야에서 활용될 수 있기를 기대합니다.
 이 프로젝트를 통해 기술의 사회적 적용 가능성을 탐구하고, 실제로 사람들의 삶에 긍정적인 변화를 가져올 수 있는 방법을 모색한 것에 대해 큰 자부심을 느낍니다. PoseNet을 활용한 자세 인식 개발은 기술이 인간의 삶을 어떻게 향상시킬 수 있는지를 보여주는 또 하나의 사례라고 생각합니다. 
