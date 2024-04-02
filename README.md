# graphics

airplane : 지형 만들기 및 작품 과제 코드입니다

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

