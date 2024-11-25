## ZenGameDev game loop implementation demonstration
# Game Loop and Asset Loading Implementation Guide

## 1. Synchronous Game Loop Implementation

A synchronous game loop is the most basic implementation, but it can block the main thread and cause performance issues.

```javascript
let running = false;
let lastTime = Date.now();
const gameState = {
    position: { x: 0, y: 0 },
    velocity: { x: 100, y: 50 }
};

function syncGameLoop() {
    while (running) {
        const currentTime = Date.now();
        const deltaTime = currentTime - lastTime;

        // Update game state
        update(deltaTime);
        
        // Render frame
        render();
        
        lastTime = currentTime;
    }
}

// Start/Stop functions
function startGame() {
    running = true;
    syncGameLoop();
}

function stopGame() {
    running = false;
}
```

### Problems with Synchronous Implementation:
- Blocks the main thread completely
- No control over frame rate
- Can freeze the browser UI
- Poor performance and high CPU usage
- No time for other browser tasks

## 2. Basic Asynchronous Game Loop

The first step in improving the game loop is to make it asynchronous using `setTimeout` or `setInterval`.

```javascript
function basicAsyncGameLoop() {
    if (!running) return;
    
    const currentTime = Date.now();
    const deltaTime = currentTime - lastTime;
    
    update(deltaTime);
    render();
    
    lastTime = currentTime;
    
    // Schedule next frame
    setTimeout(basicAsyncGameLoop, 16); // Approximately 60 FPS
}
```

### Improvements:
- Doesn't block the main thread
- Allows other browser tasks to execute
- Basic frame rate control
- More responsive UI

### Limitations:
- Not synchronized with browser's refresh rate
- Can have timing inconsistencies
- May skip frames or cause stuttering

## 3. Modern Asynchronous Game Loop with requestAnimationFrame

The recommended approach uses `requestAnimationFrame` for optimal performance and browser synchronization.

```javascript
function modernGameLoop(timestamp) {
    if (!running) return;
    
    // Convert timestamp to seconds for easier physics calculations
    const deltaTime = (timestamp - lastTime) / 1000;
    
    update(deltaTime);
    render();
    
    lastTime = timestamp;
    requestAnimationFrame(modernGameLoop);
}

// Start the game loop
function startModernGame() {
    running = true;
    lastTime = performance.now();
    requestAnimationFrame(modernGameLoop);
}
```

### Benefits:
- Synchronized with browser's refresh rate
- Better performance and smoother animation
- Automatically pauses when tab is inactive
- More accurate timing
- Power efficient
- Built-in throttling

## 4. Asynchronous Asset Loading

Modern games require efficient loading of various assets. Here's how to handle them using Promises and async/await.

### 4.1 Image Loading

```javascript
function loadImage(url) {
    return new Promise((resolve, reject) => {
        const img = new Image();
        img.onload = () => resolve(img);
        img.onerror = () => reject(new Error(`Failed to load image: ${url}`));
        img.src = url;
    });
}

// Usage with async/await
async function loadGameImages() {
    try {
        const [playerSprite, backgroundImage] = await Promise.all([
            loadImage('player.png'),
            loadImage('background.jpg')
        ]);
        return { playerSprite, backgroundImage };
    } catch (error) {
        console.error('Failed to load images:', error);
        throw error;
    }
}
```

### 4.2 Sound Loading

```javascript
function loadSound(url) {
    return new Promise((resolve, reject) => {
        const audio = new Audio();
        audio.oncanplaythrough = () => resolve(audio);
        audio.onerror = () => reject(new Error(`Failed to load sound: ${url}`));
        audio.src = url;
    });
}

// Usage with async/await
async function loadGameSounds() {
    try {
        const [backgroundMusic, jumpSound] = await Promise.all([
            loadSound('music.mp3'),
            loadSound('jump.wav')
        ]);
        return { backgroundMusic, jumpSound };
    } catch (error) {
        console.error('Failed to load sounds:', error);
        throw error;
    }
}
```

### 4.3 Complete Asset Loading System

```javascript
class AssetLoader {
    constructor() {
        this.assets = new Map();
        this.loadingPromises = new Map();
    }

    async loadAssets(assetManifest) {
        const loadingPromises = assetManifest.map(async (asset) => {
            try {
                let loaded;
                switch (asset.type) {
                    case 'image':
                        loaded = await loadImage(asset.url);
                        break;
                    case 'sound':
                        loaded = await loadSound(asset.url);
                        break;
                    default:
                        throw new Error(`Unknown asset type: ${asset.type}`);
                }
                this.assets.set(asset.id, loaded);
                return { id: asset.id, status: 'loaded' };
            } catch (error) {
                return { id: asset.id, status: 'error', error };
            }
        });

        return Promise.all(loadingPromises);
    }

    getAsset(id) {
        if (!this.assets.has(id)) {
            throw new Error(`Asset not found: ${id}`);
        }
        return this.assets.get(id);
    }
}

// Usage example
const assetLoader = new AssetLoader();
const assetManifest = [
    { id: 'player', type: 'image', url: 'player.png' },
    { id: 'background', type: 'image', url: 'background.jpg' },
    { id: 'music', type: 'sound', url: 'music.mp3' }
];

async function initializeGame() {
    try {
        // Show loading screen
        showLoadingScreen();
        
        // Load all assets
        await assetLoader.loadAssets(assetManifest);
        
        // Hide loading screen and start game
        hideLoadingScreen();
        startGame();
    } catch (error) {
        showErrorScreen(error);
    }
}
```
