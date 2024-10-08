## GamePanel.java
    
    
    package main;
    
    import javax.swing.*;
    import java.awt.*;
    import java.awt.event.*;
    
    public class GamePanel extends JPanel implements ActionListener, MouseMotionListener, MouseListener, KeyListener {
    
        private boolean play = false;
        private boolean start = false;
        private boolean gameOver = false;
        private int score = 0;
        private int totalBricks = 21;
        private Timer timer;
        private int delay = 8;
    
        private int playerX = 310;  // 패들의 초기 X 위치
        private int ballPosX = playerX + 40;  // 공의 초기 X 위치 (패들 바로 위에)
        private int ballPosY = 530;  // 공의 초기 Y 위치 (패들 바로 위에)
        private int ballDirX = 0;  // 공의 초기 방향은 0으로 설정 (패드에서 움직이기 전까지)
        private int ballDirY = 0;  // 공의 초기 방향은 0으로 설정 (패드에서 움직이기 전까지)
        private int ballSpeed = 2;  // 공의 기본 속도
    
        private boolean paddleClicked = false;  // 패들 클릭 여부
        private MapGenerator map;
        private int stage = 1;
    
        public GamePanel() {
            map = new MapGenerator(3, 7);
            addMouseMotionListener(this);
            addMouseListener(this);
            setFocusable(true);
            setFocusTraversalKeysEnabled(false);
            addKeyListener(this);
            timer = new Timer(delay, this);
            timer.start();
            setPreferredSize(new Dimension(700, 600));
        }
    
        @Override
        public void paint(Graphics g) {
            // 배경
            g.setColor(Color.black);
            g.fillRect(1, 1, 692, 592);
    
            // 벽돌 그리기
            map.draw((Graphics2D) g);
    
            // 점수 표시
            g.setColor(Color.white);
            g.setFont(new Font("돋움", Font.BOLD, 25));
            g.drawString("점수: " + score, 590, 30);
            g.drawString("스테이지: " + stage, 450, 30);
    
            // 패들 그리기
            g.setColor(Color.green);
            g.fillRect(playerX, 550, 100, 8);
    
            // 공 그리기
            g.setColor(Color.yellow);
            g.fillOval(ballPosX, ballPosY, 20, 20);
    
            // 시작 화면 메시지
            if (!start) {
                g.setColor(Color.white);
                g.setFont(new Font("돋움", Font.BOLD, 20));
                g.drawString("벽돌깨기 게임 시작하려면 패드(바)를 클릭하시오", 100, 250);
            }
    
            // 승리 메시지
            if (totalBricks <= 0) {
                play = false;
                ballDirX = 0;
                ballDirY = 0;
                g.setColor(Color.red);
                g.setFont(new Font("돋움", Font.BOLD, 30));
                g.drawString("클리어!", 260, 300);
                g.drawString("다음 스테이지: 클릭하여 진행", 230, 350);
                gameOver = false;
            }
    
            // 패배 메시지
            if (ballPosY > 570) {
                play = false;
                ballDirX = 0;
                ballDirY = 0;
                g.setColor(Color.red);
                g.setFont(new Font("돋움", Font.BOLD, 30));
                g.drawString("게임 종료.. 점수는: " + score, 190, 300);
                g.drawString("다시하기: 클릭하여 다시 시작", 230, 350);
                gameOver = true;
            }
    
            g.dispose();
        }
    
        @Override
        public void actionPerformed(ActionEvent e) {
            timer.start();
    
            if (play) {
                // 공이 패들과 충돌하는지 확인
                if (new Rectangle(ballPosX, ballPosY, 20, 20).intersects(new Rectangle(playerX, 550, 100, 8))) {
                    // 패들의 위치에 따라 공의 방향을 조정
                    int paddleCenter = playerX + 50;
                    int ballCenter = ballPosX + 10;
                    int deltaX = ballCenter - paddleCenter;
    
                    ballDirY = -ballDirY;
                    ballDirX = deltaX / 10; // 패들의 위치에 따라 공의 방향 조정
                }
    
                A: for (int i = 0; i < map.map.length; i++) {
                    for (int j = 0; j < map.map[0].length; j++) {
                        if (map.map[i][j] > 0) {
                            int brickX = j * map.brickWidth + 80;
                            int brickY = i * map.brickHeight + 50;
                            int brickWidth = map.brickWidth;
                            int brickHeight = map.brickHeight;
    
                            Rectangle rect = new Rectangle(brickX, brickY, brickWidth, brickHeight);
                            Rectangle ballRect = new Rectangle(ballPosX, ballPosY, 20, 20);
    
                            if (ballRect.intersects(rect)) {
                                map.setBrickValue(0, i, j);
                                totalBricks--;
                                score += 1;
    
                                if (ballPosX + 19 <= rect.x || ballPosX + 1 >= rect.x + rect.width) {
                                    ballDirX = -ballDirX;
                                } else {
                                    ballDirY = -ballDirY;
                                }
    
                                break A;
                            }
                        }
                    }
                }
    
                ballPosX += ballDirX * ballSpeed;
                ballPosY += ballDirY * ballSpeed;
    
                if (ballPosX < 0) {
                    ballDirX = -ballDirX;
                }
    
                if (ballPosY < 0) {
                    ballDirY = -ballDirY;
                }
    
                if (ballPosX > 670) {
                    ballDirX = -ballDirX;
                }
    
                ballSpeed = 1 + score / 10;
    
                repaint();
            }
        }
    
        @Override
        public void mouseClicked(MouseEvent e) {
            // 패드를 클릭하면 게임 시작
            if (!play && !start && new Rectangle(playerX, 550, 100, 8).contains(e.getPoint())) {
                paddleClicked = true;
                startGame();  // 게임 시작
            }
    
            // 게임 오버 상태에서 클릭하면 게임 리셋
            if (gameOver) {
                resetGame();
            }
    
            // 승리 상태에서 클릭하면 다음 스테이지로 넘어감
            if (totalBricks <= 0) {
                stage++;
                resetGame();
            }
        }
    
        @Override
        public void mouseMoved(MouseEvent e) {
            if (paddleClicked && !play) {
                // 게임이 시작되기 전에는 공이 패드 위에서 패들과 함께 움직이게 함
                playerX = e.getX() - 50;
                if (playerX < 0) {
                    playerX = 0;
                }
                if (playerX > 600) {
                    playerX = 600;
                }
                ballPosX = playerX + 40;  // 공의 위치도 패들 위에서 같이 이동
                repaint();
            }
    
            if (paddleClicked && play) {
                // 게임이 시작된 후에는 패들만 이동
                playerX = e.getX() - 50;
                if (playerX < 0) {
                    playerX = 0;
                }
                if (playerX > 600) {
                    playerX = 600;
                }
                repaint();
            }
        }
    
        @Override
        public void mouseDragged(MouseEvent e) {}
    
        @Override
        public void mousePressed(MouseEvent e) {}
    
        @Override
        public void mouseReleased(MouseEvent e) {}
    
        @Override
        public void mouseEntered(MouseEvent e) {}
    
        @Override
        public void mouseExited(MouseEvent e) {}
    
        @Override
        public void keyReleased(KeyEvent e) {}
    
        @Override
        public void keyTyped(KeyEvent e) {}
    
        @Override
        public void keyPressed(KeyEvent e) {}
    
        private void resetGame() {
            play = false;
            gameOver = false;
            start = false;
            ballPosX = playerX + 40;  // 공의 위치를 다시 패들 위로 초기화
            ballPosY = 530;  // 패들 바로 위로 초기화
            ballDirX = 0;  // 공은 움직이지 않음
            ballDirY = 0;  // 공은 움직이지 않음
            playerX = 310;
            score = 0;
            paddleClicked = false;
            
            // 스테이지에 따라 벽돌 수와 배열 설정
            switch (stage) {
                case 1:
                    totalBricks = 21;
                    map = new MapGenerator(3, 7);
                    break;
                case 2:
                    totalBricks = 28;
                    map = new MapGenerator(4, 7);
                    break;
                case 3:
                    totalBricks = 36;
                    map = new MapGenerator(6, 6);
                    break;
                case 4:
                    totalBricks = 45;
                    map = new MapGenerator(7, 7);
                    break;
                case 5:
                    totalBricks = 54;
                    map = new MapGenerator(9, 6); // 더 많은 벽돌
                    break;
                case 6:
                    totalBricks = 63;
                    map = new MapGenerator(7, 9); // 벽돌 배열 변화
                    break;
                case 7:
                    totalBricks = 72;
                    map = new MapGenerator(8, 9); // 벽돌 수 증가
                    break;
                case 8:
                    totalBricks = 81;
                    map = new MapGenerator(9, 9); // 벽돌 수 증가
                    break;
                case 9:
                    totalBricks = 90;
                    map = new MapGenerator(10, 9); // 벽돌 수 증가
                    break;
                case 10:
                    totalBricks = 100;
                    map = new MapGenerator(10, 10); // 벽돌 수 증가
                    break;
                default:
                    totalBricks = 0; // 모든 스테이지 클리어
                    break;
            }
    
            repaint();
        }
    
        private void startGame() {
            play = true;  // 게임 플레이 상태로 전환
            start = true;  // 게임 시작 상태로 전환
            ballDirX = -1;  // 공이 움직이기 시작
            ballDirY = -2;  // 공이 패드를 떠나서 움직이기 시작
            repaint();  // 게임 화면 다시 그리기
        }
    }
