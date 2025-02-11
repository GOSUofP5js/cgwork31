# cgwork31

    ////////////////////////////////////////////////////////////
    // Headers
    ////////////////////////////////////////////////////////////
    #include <SFML/Graphics.hpp>
    #include <SFML/Audio.hpp>
    #include <cmath>
    #include <ctime>
    #include <cstdlib>
    
    #ifdef SFML_SYSTEM_IOS
    #include <SFML/Main.hpp>
    #endif
    
    std::string resourcesDir()
    {
    #ifdef SFML_SYSTEM_IOS
        return "";
    #else
        return "resources/";
    #endif
    }
    
    ////////////////////////////////////////////////////////////
    /// Entry point of application
    ///
    /// \return Application exit code
    ///
    ////////////////////////////////////////////////////////////
    int main()
    {
        std::srand(static_cast<unsigned int>(std::time(NULL)));
    
        // Define some constants
        const float pi = 3.14159f;
        const float gameWidth = 800;
        const float gameHeight = 600;
        sf::Vector2f paddleSize(25, 100);
        float ballRadius = 10.f;
    
        // Create the window of the application
        sf::RenderWindow window(sf::VideoMode(static_cast<unsigned int>(gameWidth), static_cast<unsigned int>(gameHeight), 32), "SFML Tennis",
            sf::Style::Titlebar | sf::Style::Close);
        window.setVerticalSyncEnabled(true);
    
        // Load the background texture
        sf::Texture backgroundTexture;
        if (!backgroundTexture.loadFromFile(resourcesDir() + "background.png"))
            return EXIT_FAILURE;
    
        sf::Sprite background;
        background.setTexture(backgroundTexture);
        background.setScale(gameWidth / backgroundTexture.getSize().x, gameHeight / backgroundTexture.getSize().y);
    
        // Load the sounds used in the game
        sf::SoundBuffer ballSoundBuffer;
        if (!ballSoundBuffer.loadFromFile(resourcesDir() + "ball.wav"))
            return EXIT_FAILURE;
        sf::Sound ballSound(ballSoundBuffer);
    
        // Create the SFML logo texture:
        sf::Texture sfmlLogoTexture;
        if (!sfmlLogoTexture.loadFromFile(resourcesDir() + "sfml_logo.png"))
            return EXIT_FAILURE;
        sf::Sprite sfmlLogo;
        sfmlLogo.setTexture(sfmlLogoTexture);
        sfmlLogo.setPosition(170, 50);
    
        // Create the left paddle
        sf::RectangleShape leftPaddle;
        leftPaddle.setSize(paddleSize - sf::Vector2f(3, 3));
        leftPaddle.setOutlineThickness(3);
        leftPaddle.setOutlineColor(sf::Color::Black);
        leftPaddle.setFillColor(sf::Color(100, 100, 200));
        leftPaddle.setOrigin(paddleSize / 2.f);
    
        // Create the right paddle
        sf::RectangleShape rightPaddle;
        rightPaddle.setSize(paddleSize - sf::Vector2f(3, 3));
        rightPaddle.setOutlineThickness(3);
        rightPaddle.setOutlineColor(sf::Color::Black);
        rightPaddle.setFillColor(sf::Color(200, 100, 100));
        rightPaddle.setOrigin(paddleSize / 2.f);
    
        // Create the ball
        sf::CircleShape ball;
        ball.setRadius(ballRadius - 3);
        ball.setOutlineThickness(2);
        ball.setOutlineColor(sf::Color::Black);
        ball.setFillColor(sf::Color::White);
        ball.setOrigin(ballRadius / 2, ballRadius / 2);
    
            sf::CircleShape secondBall;
            secondBall.setRadius(ballRadius - 3);
            secondBall.setOutlineThickness(2);
            secondBall.setOutlineColor(sf::Color::Black);
            secondBall.setFillColor(sf::Color::Red); // 공의 색깔을 변경할 수 있습니다.
            secondBall.setOrigin(ballRadius / 2, ballRadius / 2);
            secondBall.setPosition(gameWidth / 3, gameHeight / 2); // 공의 초기 위치를 변경할 수 있습니다.
    
    
        // Load the text font
        sf::Font font;
        if (!font.loadFromFile(resourcesDir() + "tuffy.ttf"))
            return EXIT_FAILURE;
    
        // Initialize the pause message
        sf::Text pauseMessage;
        pauseMessage.setFont(font);
        pauseMessage.setCharacterSize(40);
        pauseMessage.setPosition(170.f, 200.f);
        pauseMessage.setFillColor(sf::Color::White);
    
    #ifdef SFML_SYSTEM_IOS
        pauseMessage.setString("Welcome to SFML Tennis!\nTouch the screen to start the game.");
    #else
        pauseMessage.setString("Welcome to SFML Tennis!\n\nPress space to start the game.");
    #endif
    
        // Initialize the score display
        sf::Text scoreMessage;
        scoreMessage.setFont(font);
        scoreMessage.setCharacterSize(30);
        scoreMessage.setPosition(gameWidth / 2.25, 10.f);
        scoreMessage.setFillColor(sf::Color::White);
        int playerScore = 0;
        int computerScore = 0;
    
        // Define the paddles properties
        sf::Clock AITimer;
        const sf::Time AITime = sf::seconds(0.1f);
        const float paddleSpeed = 400.f;
        float rightPaddleSpeed = 0.f;
        const float ballSpeed = 400.f;
        float ballAngle = 0.f; // to be changed later
    
        sf::Clock clock;
        bool isPlaying = false;
        bool isGameOver = false;
        while (window.isOpen())
        {
            // Handle events
            sf::Event event;
            while (window.pollEvent(event))
            {
                // Window closed or escape key pressed: exit
                if ((event.type == sf::Event::Closed) ||
                    ((event.type == sf::Event::KeyPressed) && (event.key.code == sf::Keyboard::Escape)))
                {
                    window.close();
                    break;
                }
    
                // Space key pressed: play
                if (((event.type == sf::Event::KeyPressed) && (event.key.code == sf::Keyboard::Space)) ||
                    (event.type == sf::Event::TouchBegan))
                {
                    if (!isPlaying && !isGameOver)
                    {
                        // (re)start the game
                        isPlaying = true;
                        clock.restart();
    
                        // Reset the position of the paddles and ball
                        leftPaddle.setPosition(10.f + paddleSize.x / 2.f, gameHeight / 2.f);
                        rightPaddle.setPosition(gameWidth - 10.f - paddleSize.x / 2.f, gameHeight / 2.f);
                        ball.setPosition(gameWidth / 2.f, gameHeight / 2.f);
    
                        // Reset the ball angle
                        do
                        {
                            // Make sure the ball initial angle is not too much vertical
                            ballAngle = static_cast<float>(std::rand() % 360) * 2.f * pi / 360.f;
                        } while (std::abs(std::cos(ballAngle)) < 0.7f);
                    }
                    else if (isGameOver)
                    {
                        // Reset the game
                        isGameOver = false;
                        playerScore = 0;
                        computerScore = 0;
                        pauseMessage.setString("Welcome to SFML Tennis!\n\nPress space to start the game.");
                    }
                }
    
                // Window size changed, adjust view appropriately
                if (event.type == sf::Event::Resized)
                {
                    sf::View view;
                    view.setSize(gameWidth, gameHeight);
                    view.setCenter(gameWidth / 2.f, gameHeight / 2.f);
                    window.setView(view);
                }
            }
    
            if (isPlaying)
            {
                float deltaTime = clock.restart().asSeconds();
    
                // Move the player's paddle
                if (sf::Keyboard::isKeyPressed(sf::Keyboard::Up) &&
                    (leftPaddle.getPosition().y - paddleSize.y / 2 > 5.f))
                {
                    leftPaddle.move(0.f, -paddleSpeed * deltaTime);
                }
                if (sf::Keyboard::isKeyPressed(sf::Keyboard::Down) &&
                    (leftPaddle.getPosition().y + paddleSize.y / 2 < gameHeight - 5.f))
                {
                    leftPaddle.move(0.f, paddleSpeed * deltaTime);
                }
    
                if (sf::Touch::isDown(0))
                {
                    sf::Vector2i pos = sf::Touch::getPosition(0);
                    sf::Vector2f mappedPos = window.mapPixelToCoords(pos);
                    leftPaddle.setPosition(leftPaddle.getPosition().x, mappedPos.y);
                }
    
                // Move the computer's paddle
                if (((rightPaddleSpeed < 0.f) && (rightPaddle.getPosition().y - paddleSize.y / 2 > 5.f)) ||
                    ((rightPaddleSpeed > 0.f) && (rightPaddle.getPosition().y + paddleSize.y / 2 < gameHeight - 5.f)))
                {
                    rightPaddle.move(0.f, rightPaddleSpeed * deltaTime);
                }
    
                // Update the computer's paddle direction according to the ball position
                if (AITimer.getElapsedTime() > AITime)
                {
                    AITimer.restart();
                    if (ball.getPosition().y + ballRadius > rightPaddle.getPosition().y + paddleSize.y / 2)
                        rightPaddleSpeed = paddleSpeed;
                    else if (ball.getPosition().y - ballRadius < rightPaddle.getPosition().y - paddleSize.y / 2)
                        rightPaddleSpeed = -paddleSpeed;
                    else
                        rightPaddleSpeed = 0.f;
                }
    
                // Move the ball
                float factor = ballSpeed * deltaTime;
                ball.move(std::cos(ballAngle) * factor, std::sin(ballAngle) * factor);
    
    #ifdef SFML_SYSTEM_IOS
                const std::string inputString = "Touch the screen to restart.";
    #else
                const std::string inputString = "Press space to restart or\nescape to exit.";
    #endif
    
                // Check collisions between the ball and the screen
                if (ball.getPosition().x - ballRadius < 0.f)
                {
                    computerScore++;
                    if (computerScore >= 5)
                    {
                        isPlaying = false;
                        isGameOver = true;
                        pauseMessage.setString("You Lost the Game!\n\n" + inputString);
                    }
                    else
                    {
                        ball.setPosition(gameWidth / 2.f, gameHeight / 2.f);
                        do
                        {
                            ballAngle = static_cast<float>(std::rand() % 360) * 2.f * pi / 360.f;
                        } while (std::abs(std::cos(ballAngle)) < 0.7f);
                    }
                }
    
                    if (secondBall.getPosition().x - ballRadius < leftPaddle.getPosition().x + paddleSize.x / 2 &&
                        secondBall.getPosition().x - ballRadius > leftPaddle.getPosition().x &&
                        secondBall.getPosition().y + ballRadius >= leftPaddle.getPosition().y - paddleSize.y / 2 &&
                        secondBall.getPosition().y - ballRadius <= leftPaddle.getPosition().y + paddleSize.y / 2)
                    {
                        // 왼쪽 패들과의 충돌 처리
                    }
    
                    // 오른쪽 패들과의 충돌 검사
                    if (secondBall.getPosition().x + ballRadius > rightPaddle.getPosition().x - paddleSize.x / 2 &&
                        secondBall.getPosition().x + ballRadius < rightPaddle.getPosition().x &&
                        secondBall.getPosition().y + ballRadius >= rightPaddle.getPosition().y - paddleSize.y / 2 &&
                        secondBall.getPosition().y - ballRadius <= rightPaddle.getPosition().y + paddleSize.y / 2)
                    {
                        // 오른쪽 패들과의 충돌 처리
                    }
    
                // Move the second ball
                float secondBallAngle = 45.f * pi / 180.f; // 이동 각도 설정 (45도)
                float secondBallSpeed = 300.f; // 이동 속도 설정
    
                // 두 번째 공 이동 처리
                float secondBallFactor = secondBallSpeed * deltaTime;
                secondBall.move(std::cos(secondBallAngle)* secondBallFactor, std::sin(secondBallAngle)* secondBallFactor);
    
                // Check collisions between the second ball and the screen boundaries
                // 두 번째 공과 화면 경계 간의 충돌 확인
                if (secondBall.getPosition().x - ballRadius < 0.f || secondBall.getPosition().x + ballRadius > gameWidth)
                {
                    // 화면 좌우 경계에 도달하면 이동 방향을 반전시킵니다.
                    secondBallAngle = -secondBallAngle;
                }
                if (secondBall.getPosition().y - ballRadius < 0.f || secondBall.getPosition().y + ballRadius > gameHeight)
                {
                    // 화면 상하 경계에 도달하면 이동 방향을 반전시킵니다.
                    secondBallAngle = pi - secondBallAngle;
                }
    
                if (ball.getPosition().x + ballRadius > gameWidth)
                {
                    playerScore++;
                    if (playerScore >= 5)
                    {
                        isPlaying = false;
                        isGameOver = true;
                        pauseMessage.setString("You Won the Game!\n\n" + inputString);
                    }
                    else
                    {
                        ball.setPosition(gameWidth / 2.f, gameHeight / 2.f);
                        do
                        {
                            ballAngle = static_cast<float>(std::rand() % 360) * 2.f * pi / 360.f;
                        } while (std::abs(std::cos(ballAngle)) < 0.7f);
                    }
                }
                if (ball.getPosition().y - ballRadius < 0.f)
                {
                    ballSound.play();
                    ballAngle = -ballAngle;
                    ball.setPosition(ball.getPosition().x, ballRadius + 0.1f);
                }
                if (ball.getPosition().y + ballRadius > gameHeight)
                {
                    ballSound.play();
                    ballAngle = -ballAngle;
                    ball.setPosition(ball.getPosition().x, gameHeight - ballRadius - 0.1f);
                }
    
                // Check the collisions between the ball and the paddles
                // Left Paddle
                if (ball.getPosition().x - ballRadius < leftPaddle.getPosition().x + paddleSize.x / 2 &&
                    ball.getPosition().x - ballRadius > leftPaddle.getPosition().x &&
                    ball.getPosition().y + ballRadius >= leftPaddle.getPosition().y - paddleSize.y / 2 &&
                    ball.getPosition().y - ballRadius <= leftPaddle.getPosition().y + paddleSize.y / 2)
                {
                    if (ball.getPosition().y > leftPaddle.getPosition().y)
                        ballAngle = pi - ballAngle + static_cast<float>(std::rand() % 20) * pi / 180;
                    else
                        ballAngle = pi - ballAngle - static_cast<float>(std::rand() % 20) * pi / 180;
    
                    ballSound.play();
                    ball.setPosition(leftPaddle.getPosition().x + ballRadius + paddleSize.x / 2 + 0.1f, ball.getPosition().y);
                }
    
                // Right Paddle
                if (ball.getPosition().x + ballRadius > rightPaddle.getPosition().x - paddleSize.x / 2 &&
                    ball.getPosition().x + ballRadius < rightPaddle.getPosition().x &&
                    ball.getPosition().y + ballRadius >= rightPaddle.getPosition().y - paddleSize.y / 2 &&
                    ball.getPosition().y - ballRadius <= rightPaddle.getPosition().y + paddleSize.y / 2)
                {
                    if (ball.getPosition().y > rightPaddle.getPosition().y)
                        ballAngle = pi - ballAngle + static_cast<float>(std::rand() % 20) * pi / 180;
                    else
                        ballAngle = pi - ballAngle - static_cast<float>(std::rand() % 20) * pi / 180;
    
                    ballSound.play();
                    ball.setPosition(rightPaddle.getPosition().x - ballRadius - paddleSize.x / 2 - 0.1f, ball.getPosition().y);
                }
            }
    
                // Clear the window
                window.clear(sf::Color(50, 50, 50));
    
                // Draw the background
                window.draw(background);
    
                if (isPlaying)
                {
                    // Draw the paddles and the ball
                    window.draw(leftPaddle);
                    window.draw(rightPaddle);
                    window.draw(ball);
                    window.draw(secondBall);
                }
                else
                {
                    // Draw the pause message
                    window.draw(pauseMessage);
                    window.draw(sfmlLogo);
                }
    
                // Update and draw the score
                scoreMessage.setString(std::to_string(playerScore) + " vs " + std::to_string(computerScore));
                window.draw(scoreMessage);
    
                // Display things on screen
                window.display();
        }
    
        return EXIT_SUCCESS;
    }

