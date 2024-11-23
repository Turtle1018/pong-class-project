package io.github.project_1;

import com.badlogic.gdx.ApplicationAdapter;
import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.Input;
import com.badlogic.gdx.graphics.GL20;
import com.badlogic.gdx.graphics.OrthographicCamera;
import com.badlogic.gdx.graphics.glutils.ShapeRenderer;
import com.badlogic.gdx.graphics.Color;
import com.badlogic.gdx.graphics.g2d.BitmapFont;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import java.util.Random;

public class Main extends ApplicationAdapter {
    private OrthographicCamera camera;
    private ShapeRenderer shapeRenderer;
    private SpriteBatch spriteBatch;
    private BitmapFont font;

    // Game constants
    private static final int SCREEN_HEIGHT = 600;
    private static final int PADDLE_WIDTH = 10;
    private static final int PADDLE_HEIGHT = 100;
    private static final int BALL_SIZE = 10;
    private static final int SCREEN_WIDTH = 1200;

    // Paddle positions
    private float paddle1Y, paddle2Y;
    private static final float PADDLE_SPEED = 300;

    // Ball position and velocity
    private float ballX, ballY, ballVelocityX, ballVelocityY;
    private boolean ballStuckToPaddle = false; // Flag to indicate if the ball is stuck
    private boolean stickToPaddle1 = true;     // Determines which paddle the ball sticks to

    // Scores
    private int player1Score = 0;
    private int player2Score = 0;

    // Initial score limit
    private int scoreLimit = 12;

    // Game state
    private boolean isGameOver = false;
    private boolean isGameStarted = false; // New: Flag for start-up screen

    // Timer for speed increase
    private float pointTimer = 0;
    private static final float POINT_TIMEOUT = 10f; // 10 seconds
    private static final float SPEED_MULTIPLIER = 2.5f; // Speed increase factor

    @Override
    public void create() {
        camera = new OrthographicCamera(SCREEN_WIDTH, SCREEN_HEIGHT);
        camera.setToOrtho(false, SCREEN_WIDTH, SCREEN_HEIGHT);
        shapeRenderer = new ShapeRenderer();
        spriteBatch = new SpriteBatch();
        font = new BitmapFont();
        font.setColor(Color.WHITE);
        font.getData().setScale(2); // Scale up the font size

        // Initialize paddles
        paddle1Y = (SCREEN_HEIGHT - PADDLE_HEIGHT) / 2;
        paddle2Y = (SCREEN_HEIGHT - PADDLE_HEIGHT) / 2;

        // Initialize ball
        resetBall();
    }

    @Override
    public void render() {
        // Clear the screen
        Gdx.gl.glClearColor(0, 0, 0, 1);
        Gdx.gl.glClear(GL20.GL_COLOR_BUFFER_BIT);

        if (!isGameStarted) {
            renderStartScreen(); // Show start-up screen
            if (Gdx.input.isKeyJustPressed(Input.Keys.ENTER)) {
                isGameStarted = true; // Start the game
            }
            return; // Skip the rest of the game logic
        }

        if (!isGameOver) {
            update(Gdx.graphics.getDeltaTime());
        } else {
            checkRestart(); // Check if the user wants to restart
        }

        // Render game elements
        shapeRenderer.setProjectionMatrix(camera.combined);
        shapeRenderer.begin(ShapeRenderer.ShapeType.Filled);

        // Draw paddles
        shapeRenderer.rect(10, paddle1Y, PADDLE_WIDTH, PADDLE_HEIGHT);
        shapeRenderer.rect(SCREEN_WIDTH - 20, paddle2Y, PADDLE_WIDTH, PADDLE_HEIGHT);

        // Draw ball
        shapeRenderer.rect(ballX, ballY, BALL_SIZE, BALL_SIZE);

        shapeRenderer.end();

        // Draw scores and game over message
        spriteBatch.setProjectionMatrix(camera.combined);
        spriteBatch.begin();
        drawScores();
        if (isGameOver) {
            drawGameOverMessage();
        }
        spriteBatch.end();
    }

    private void renderStartScreen() {
        spriteBatch.setProjectionMatrix(camera.combined);
        spriteBatch.begin();
        font.draw(spriteBatch, "PONG", SCREEN_WIDTH / 2 - 50, SCREEN_HEIGHT / 2 + 100);
        font.draw(spriteBatch, "Press ENTER to Start", SCREEN_WIDTH / 2 - 150, SCREEN_HEIGHT / 2);
        spriteBatch.end();
    }

    private void update(float deltaTime) {
        // Update the point timer
        pointTimer += deltaTime;

        // If 10 seconds pass without a point being scored, increase speed
        if (pointTimer >= POINT_TIMEOUT) {
            ballVelocityX *= SPEED_MULTIPLIER;
            ballVelocityY *= SPEED_MULTIPLIER;
            pointTimer = 0; // Reset timer after speed increase
        }

        // Paddle 1 movement
        if (Gdx.input.isKeyPressed(Input.Keys.W)) {
            paddle1Y += PADDLE_SPEED * deltaTime;
        }
        if (Gdx.input.isKeyPressed(Input.Keys.S)) {
            paddle1Y -= PADDLE_SPEED * deltaTime;
        }

        // Paddle 2 movement
        if (Gdx.input.isKeyPressed(Input.Keys.UP)) {
            paddle2Y += PADDLE_SPEED * deltaTime;
        }
        if (Gdx.input.isKeyPressed(Input.Keys.DOWN)) {
            paddle2Y -= PADDLE_SPEED * deltaTime;
        }

        // Clamp paddles to screen bounds
        paddle1Y = Math.max(0, Math.min(SCREEN_HEIGHT - PADDLE_HEIGHT, paddle1Y));
        paddle2Y = Math.max(0, Math.min(SCREEN_HEIGHT - PADDLE_HEIGHT, paddle2Y));

        // Ball behavior when stuck
        if (ballStuckToPaddle) {
            stickBallToPaddle();
            if (Gdx.input.isKeyJustPressed(Input.Keys.SPACE)) {
                releaseBall();
            }
            return;
        }

        // Update ball position
        ballX += ballVelocityX * deltaTime;
        ballY += ballVelocityY * deltaTime;

        // Ball collision with top and bottom walls
        if (ballY <= 0 || ballY + BALL_SIZE >= SCREEN_HEIGHT) {
            ballVelocityY = -ballVelocityY;
        }

        // Ball collision with paddles
        if (ballX <= 20 && ballY + BALL_SIZE > paddle1Y && ballY < paddle1Y + PADDLE_HEIGHT) {
            ballStuckToPaddle = true;
            stickToPaddle1 = true;
        } else if (ballX + BALL_SIZE >= SCREEN_WIDTH - 20 && ballY + BALL_SIZE > paddle2Y && ballY < paddle2Y + PADDLE_HEIGHT) {
            ballStuckToPaddle = true;
            stickToPaddle1 = false;
        }

        // Check for ball going out of bounds
        if (ballX < 0) {
            player2Score++;
            checkScoreLimit();
            checkGameOver();
            resetBall();
        } else if (ballX + BALL_SIZE > SCREEN_WIDTH) {
            player1Score++;
            checkScoreLimit();
            checkGameOver();
            resetBall();
        }
    }

    private void checkScoreLimit() {
        // Increase the score limit to 13 if both players reach 11
        if (player1Score >= 11 && player2Score >= 11) {
            scoreLimit = 13;
        }
    }

    private void stickBallToPaddle() {
        if (stickToPaddle1) {
            ballX = 20; // Stick to player 1 paddle
            ballY = paddle1Y + PADDLE_HEIGHT / 2 - BALL_SIZE / 2;
        } else {
            ballX = SCREEN_WIDTH - 20 - BALL_SIZE; // Stick to player 2 paddle
            ballY = paddle2Y + PADDLE_HEIGHT / 2 - BALL_SIZE / 2;
        }
    }

    private void releaseBall() {
        Random random = new Random();
        ballVelocityX = stickToPaddle1 ? 200 : -200; // Direction based on paddle
        ballVelocityY = random.nextBoolean() ? 200 : -200; // 50/50 chance for vertical direction
        ballStuckToPaddle = false;
    }

    private void resetBall() {
        ballStuckToPaddle = true;
        stickToPaddle1 = new Random().nextBoolean(); // Randomly decide which paddle starts
        stickBallToPaddle();
        pointTimer = 0; // Reset the timer whenever the ball is reset
    }

    private void drawScores() {
        font.draw(spriteBatch, "Player 1: " + player1Score, 50, SCREEN_HEIGHT - 50);
        font.draw(spriteBatch, "Player 2: " + player2Score, SCREEN_WIDTH - 200, SCREEN_HEIGHT - 50);
    }

    private void drawGameOverMessage() {
        String winner = player1Score >= scoreLimit ? "Player 1" : "Player 2";
        font.draw(spriteBatch, "Game Over! " + winner + " Wins!", SCREEN_WIDTH / 2 - 200, SCREEN_HEIGHT / 2 + 50);
        font.draw(spriteBatch, "Press Q to Restart", SCREEN_WIDTH / 2 - 150, SCREEN_HEIGHT / 2 - 50);
    }

    private void checkGameOver() {
        if (player1Score >= scoreLimit || player2Score >= scoreLimit) {
            isGameOver = true;
        }
    }

    private void checkRestart() {
        if (Gdx.input.isKeyJustPressed(Input.Keys.Q)) {
            isGameOver = false;
            player1Score = 0;
            player2Score = 0;
            scoreLimit = 12; // Reset score limit after restart
            resetBall();
        }
    }

    @Override
    public void dispose() {
        spriteBatch.dispose();
        shapeRenderer.dispose();
        font.dispose();
    }
}
