<script>
    import kaplay from "kaplay"
    import { browser } from '$app/environment'
    import { onMount } from 'svelte'
    import { mountain_facts, forest_facts, farm_facts } from '$lib/facts'

    export let isMenu, world
    // let waveDialog

    let canvasCollection, canvasArray

    let score = 0
    let waveText = ""
    let fact
    let wave

    onMount(() => {
        canvasCollection = document.getElementsByTagName("canvas")
        canvasArray = [...canvasCollection]
        canvasArray.forEach((d,i) => {
            if(i == canvasArray.length-1) {
            } else {
                d.remove()
            }
        })
        canvasCollection = document.getElementsByTagName("canvas")
        canvasArray = [...canvasCollection]
    })


    // 
    function setPos(b,w) {
        return b/2-(3*w)
    }

    $: if (browser) {
        kaplay({
            background: [240, 220, 106],
        })

        // console.log(world)

        const BULLET_SPEED = 400
	    const ENEMY_SPEED = 120
        const OBJ_HEALTH = 2

        loadFont("Jua", "/fonts/Jua-Regular.ttf")
        loadFont("Nunito", "/fonts/Nunito-VariableFont_wght.ttf")

        loadSprite("pineapple", "/images/sprites/pineapple.png");
        loadSprite("bean", "/images/sprites/bean.png");
        loadSprite("ghosty", "/images/sprites/ghosty.png");
        loadSprite("farmGroundTile", "/images/sprites/farm-ground-tile.png");
        loadSprite("forestGroundTile", "/images/sprites/forest-ground-tile.png")
        loadSprite("mountainGroundTile", "/images/sprites/mountain-ground-tile.png")

        loadSprite("mountBlock", "/images/sprites/mountain_blocker.png");
        loadSprite("farmBlock", "/images/sprites/farm_blocker.png");
        loadSprite("forestBlock", "/images/sprites/forest_blocker.png");

        loadSprite("farmBase", "/images/sprites/farm-base-tile.png")
        loadSprite("mountainBase", "/images/sprites/mountain-base-tile.png")
        loadSprite("forestBase", "/images/sprites/forest-base-tile.png")

        loadSprite("farmBullet", "/images/sprites/farm_bullet.png")
        loadSprite("mountainBullet", "/images/sprites/mountain_bullet.png")
        loadSprite("forestBullet", "/images/sprites/forest_bullet.png")

        loadSprite("farmShoot", "/images/sprites/farm_shooter.png", {
            sliceX:6,
            anims: {
                "idle":0,
                "move":{
                    from:0,
                    to:5,
                    speed:12,
                    loop:false
                }
            }
        })
        loadSprite("mountainShoot", "/images/sprites/mountain_shooter.png", {
            sliceX:6,
            anims: {
                "idle":0,
                "move":{
                    from:0,
                    to:5,
                    speed:12,
                    loop:false
                }
            }
        })
        loadSprite("forestShoot", "/images/sprites/forest_shooter.png", {
            sliceX:6,
            anims: {
                "idle":0,
                "move":{
                    from:0,
                    to:5,
                    speed:12,
                    loop:false
                }
            }
        })

        loadSprite("forestEnemy", "/images/sprites/forest_enemy.png", {
            sliceX:4,
            anims: {
                "idle":0,
                "move":{
                    from:0,
                    to:3,
                    speed:8,
                    loop:true
                }
            }
        })
        loadSprite("farmEnemy", "/images/sprites/farm_enemy.png", {
            sliceX:4,
            anims: {
                "idle":0,
                "move":{
                    from:0,
                    to:3,
                    speed:8,
                    loop:true
                }
            }
        })
        loadSprite("mountainEnemy", "/images/sprites/mountain_enemy.png", {
            sliceX:4,
            anims: {
                "idle":0,
                "move":{
                    from:0,
                    to:3,
                    speed:8,
                    loop:true
                }
            }
        })

        loadSprite("noTex", "/images/sprites/cross.png")

        loadSound("hit", "/audio/hit.mp3");
        loadSound("explode", "/audio/explode.mp3");
        loadSound("shoot", "/audio/shoot.mp3");
        loadSound("bgMusic", "/audio/bgmusic.mp3");

        volume(0.5);

        const music = play("bgMusic", {
            loop: true,
            paused: true,
        });

        let grids = []
        let square = 144
        let side = Math.sqrt(square)
        let w = 64
        let yInitPos = 0, xInitPos = 0
        let level
        let onGame = false

        let allTowers = []

        let shooter = true

        function checkOrientation() {
            if (width() > height()) {
                // landscape
                // w = grid size
                w = height()/6*0.55
                xInitPos = setPos(width()+w,w)
                yInitPos = height()/8*2
            } else if (height() > width()) {
                // portrait
                w = width()/6*0.8
                yInitPos = setPos(height()+w,w)
                xInitPos = width()/6
            }
        }

        scene("startGame", ({world:world}) => {
            
            music.paused = false

            checkOrientation()

            onResize(() => {
                checkOrientation()
            })        

            let budget = 20
            let basehealth = 100
            let wave =[3,6,5,10,8,16]
            let n = 0, waveIndex = 0
            let d = 0
            
            onGame = false
            shooter = true

            level = addLevel([
                // Design the level layout with symbols
                "%#####",
                "%#####",
                "%#####",
                "%#####",
                "%#####",
                "%#####",

            ], {
                // The size of each grid
                tileWidth: w,
                tileHeight: w,
                // The position of the top left block
                pos: vec2(xInitPos, yInitPos),
                // Define what each symbol means (in components)
                tiles: {
                    "#": () => [
                        sprite(
                            world == "Farmland" ? "farmGroundTile" :
                            world == "Forest" ? "forestGroundTile" :
                            world == "Mountain" ? "mountainGroundTile" : "noTex"),
                        area(),
                        anchor("center"),
                        scale(w/32),
                        "grid",
                        "ground",
                        "tile",
                        {
                            tower : false,
                            name : "grid"
                        },
                    ],
                    "%": () => [
                        sprite(
                            world == "Farmland" ? "farmBase" :
                            world == "Forest" ? "forestBase" :
                            world == "Mountain" ? "mountainBase" : "noTex"),
                        area(),
                        pos(0,w/2),
                        anchor("bot"),
                        scale(w/32),
                        "base",
                        "ground",
                        "tile",
                        {
                            name : "base",
                            state: "alive"
                        },
                    ],
                },
            });

            const allItem = get("*")
            console.log(allItem)

            allItem[0].children.forEach((c, i) => {
                if (c.name == "grid") {
                    c.add([
                        rect(2,20),
                        pos(-w/4,0),
                        anchor("center"),
                        area(),
                        scale(w/64),
                        opacity(0),
                        "stopper"
                    ])
                } else if (c.name == "base") {
                    c.add([
                        rect(2,20),
                        pos(0,0),
                        anchor("center"),
                        color(YELLOW),
                        area(),
                        scale(w/64),
                        opacity(0),
                        z(20),
                        "basestopper",
                        {
                            index:i
                        }
                    ])
                }
            })

            const yPos = [...new Set(allItem[0].children.filter(d=>d.name == "grid").map(d => d.pos.y))]

            const xMax = Math.max(...allItem[0].children.filter(d=>d.name == "grid").map(d => d.pos.x))

            let cardContainer = add([
                rect(200,120),
                    anchor("center"),
                    pos(width()/2, height()/24*21),
                    opacity(0)
            ])

            let shooterCard = cardContainer.add([
                rect(80,120, {radius:4}),
                anchor("center"),
                pos(-50, 0),
                outline(2),
                scale(1.1),
                area(),
                "shootercard"
            ])
            shooterCard.add([
                text("plant", {size:16, font:"Jua"}),
                pos(0,-45),
                anchor("center"),
                color(BLACK)
            ])
            shooterCard.add([
                sprite(
                    world == "Farmland" ? "farmShoot" :
                    world == "Mountain" ? "mountainShoot" : 
                    world == "Forest" ? "forestShoot" : 
                    "pineapple"), 
                pos(0,0),
                anchor("center"),
                scale(1.5),
            ])
            shooterCard.add([
                text("cost: 7", {size:16, font:"Jua"}),
                pos(0,45),
                anchor("center"),
                color(BLACK)
            ])
            let blockerCard = cardContainer.add([
                rect(80,120, {radius:4}),
                anchor("center"),
                pos(50, 0),
                outline(2),
                scale(0.95),
                area(),
                "blockercard"
            ])
            blockerCard.add([
                text("rock", {size:16, font:"Jua"}),
                pos(0,-45),
                anchor("center"),
                color(BLACK)
            ])
            blockerCard.add([
                sprite(world == "Farmland" ? "farmBlock" : world == "Mountain" ? "mountBlock" : world == "Forest" ? "forestBlock" : "pineapple"), 
                pos(0,0),
                anchor("center"),
                scale(1.5),
            ])
            blockerCard.add([
                text("cost: 3", {size:16, font:"Jua"}),
                pos(0,45),
                anchor("center"),
                color(BLACK)
            ])

            shooterCard.onClick((c) => {
                if (onGame) {
                    shooter = true
                    blockerCard.scale = vec2(0.95)
                    shooterCard.scale = vec2(1.1)
                }
            })

            blockerCard.onClick((c) => {
                if (onGame) {
                    shooter = false
                    shooterCard.scale = vec2(0.95)
                    blockerCard.scale = vec2(1.1)
                }
            })

            let label = add([
                text(budget, {font:"Jua"}),
                pos(width()/2, 70),
                anchor("center"),
                color(BLACK)
            ])

            let worldLevel = add([
                text(`${world}`, {
                    width:300,
                    size:16,
                    align:"center",
                    font:"Nunito"
                }),
                pos(width()/2, 40),
                anchor("center"),
                color(BLACK)
            ])

            let healthBarContainer = add([
                rect(200,15),
                anchor("center"),
                pos(width()/2,100),
                outline(4)
            ])

            let healthBar = healthBarContainer.add([
                rect(basehealth*2,12),
                anchor("left"),
                pos(-100    ,0),
                color(GREEN)
            ])

            onClick("grid", (d) => {
                if (onGame) {
                    if (!d.tower) {
                        if (shooter) {
                            if (budget >= 7) {
                                d.tower = true
                                addSprite(d.id, d.pos.x + xInitPos, d.pos.y+ yInitPos, yPos.findIndex(y => y == d.pos.y), "shooter")
                                budget-=7
                                label.text = budget
                            }
                        } else if (!shooter) {
                            if (budget >= 3) {
                                d.tower = true
                                addSprite(d.id, d.pos.x + xInitPos, d.pos.y+ yInitPos, yPos.findIndex(y => y == d.pos.y), "blocker")
                                budget-=3
                                label.text = budget
                            }
                        }
                    } else {
                        console.log("tower already exist")
                    }
                }
            })

            // onAdd("tower", (d) => {
            //     shootBullet(d.grid_id, d.pos.x, d.pos.y, d.y_index)
            // })

            onAdd("enemy", (e) => {
                e.play("move")
                e.onStateEnter("idle", () => {
                    e.play("idle")
                    wait(1,() => {
                        e.enterState("move")
                        e.play("move")
                    })
                })
            })

            function addSprite(id,x,y,i, type) {
                if (type == "shooter") {
                    allTowers[id] = add([
                        sprite(
                            world == "Farmland" ? "farmShoot" :
                            world == "Mountain" ? "mountainShoot" : 
                            world == "Forest" ? "forestShoot" : 
                            "pineapple"
                        ), 
                        pos(x, y+(w/2)),
                        anchor("bot"),
                        area({ shape: new Rect(vec2(0), 24, 24) }),
                        scale(w/32),
                        z(i+10),
                        "tower",
                        state("move", ["move", "idle"]),
                        {
                            grid_id:id,
                            y_index:i,
                            blocker:false
                        }
                        ])
                    shootBullet(id, x, y+(w/2), i)
                } else if (type == "blocker") {
                    add([
                        sprite(world == "Farmland" ? "farmBlock" : world == "Mountain" ? "mountBlock" : world == "Forest" ? "forestBlock" : "pineapple"), 
                        pos(x, y+(w/2)),
                        anchor("bot"),
                        area({ shape: new Rect(vec2(0), 24, 24) }),
                        health(2),
                        scale(w/32),
                        z(i+10),
                        "blocker",
                        {
                            grid_id:id,
                            y_index:i,
                            blocker:true
                        }
                        ]);
                }
            }

            function removeSprite(id) {

            }

            function shootBullet(id,x,y,i) {
                let enemies = get("enemy").map(e => e.y_index)

                if (onGame) {
                    let selGrid = allItem[0].children.map(d => d.id).findIndex(d => d == id)
                    if (allItem[0].children[selGrid].tower) {
                        if (enemies.includes(i)) {
                            allTowers[id].play("move")
                            //shoot bullet
                            wait(0.3, () => {
                                play("shoot")
                                add([
                                    sprite(
                                        world == "Farmland" ? "farmBullet" :
                                        world == "Forest" ? "forestBullet" :
                                        world == "Mountain" ? "mountainBullet" : "pineapple"),
                                    area(),
                                    pos(x,y-(w/2)),
                                    anchor("center"),
                                    scale(w/32),
                                    move(RIGHT, BULLET_SPEED),
                                    offscreen({ destroy: true }),
                                    // strings here means a tag
                                    "bullet",
                                ])
                                wait(2, () => shootBullet(id,x,y,i))
                            })
                            
                        } else {
                            wait(1, () => shootBullet(id,x,y,i))
                        }
                    }
                }
            }

            function updateW(z) {
                let index = randi(0,yPos.length)
                if (z == 1) {
                    waveIndex++
                    addEnemy(index)
                } else {
                    waveIndex = 0
                }
            }

            let wavebox

            function showWaveText(w, t) {
                wavebox = add([
                    rect(300,180, {radius:4}),
                    pos(width()/2, height()/2),
                    anchor("center"),
                    color(WHITE),
                    area(),
                    outline(2),
                    z(100),
                    "wavebutton"
                ])

                wavebox.add([
                    text(w, {
                        size:20,
                        font:"Jua"
                    }),
                    anchor("top"),
                    pos(0,-75),
                    color(BLACK),
                    area()
                ])

                wavebox.add([
                    text(t, {
                        size:12,
                        width:250,
                        font:"Nunito"
                    }),
                    pos(0,-50),
                    anchor("top"),
                    color(BLACK),
                    area()
                ])

                wavebox.add([
                    text("click to start wave", {
                        size:12,
                        font:"Nunito"
                        }),
                    pos(0,75),
                    anchor("bot"),
                    color(BLACK),
                    area()
                ])
            
            }

            function spawnEnemy() {
                if (onGame) {
                    updateW(1) 
                    // console.log(waveIndex, wave[n])
                    wait(4, () =>{
                        if(waveIndex < wave[n]) {
                            spawnEnemy()
                        }
                    })
                }
            }

            function addEnemy(i) {
                add([
                    sprite(world == "Farmland" ? "farmEnemy" :
                            world == "Forest" ? "forestEnemy" :
                            world == "Mountain" ? "mountainEnemy" : "noTex"),
                    area({ shape: new Rect(vec2(0), 24, 24) }),
                    scale(w/32),
                    pos(xMax+width()/3+100, yPos[i]+yInitPos+(w/2)),
                    z(i+10),
                    health(n < 2 ? 2 : n >= 2 && n < 4 ? 3 : n >= 4 ? 5 : 4),
                    anchor("bot"),
                    "trash",
                    "enemy",
                    `enemy-${i}`,
                    state("move", ["move", "idle"]),
                    { 
                        speed: ENEMY_SPEED,
                        distance: 0,
                        y_index:i
                    },
                ])
            }

            function grow(rate) {
                return {
                    update() {
                        const n = rate * dt()
                        this.scale.x += n
                        this.scale.y += n
                    },
                }
            }

            function addExplode(p, n, rad, size) {
                for (let i = 0; i < n; i++) {
                    wait(rand(n * 0.1), () => {
                        for (let i = 0; i < 2; i++) {
                            add([
                                pos(p.add(rand(vec2(-rad), vec2(rad)))),
                                circle(4),
                                scale(1 * size, 1 * size),
                                lifespan(0.1),
                                grow(rand(48, 72) * size),
                                anchor("center"),
                            ])
                        }
                    })
                }
            }

            onUpdate("trash", (t) => {
                if (t.state == "move") {
                    t.move(-t.speed, 0)
                    if (t.pos.x + t.width < width()/3 - 100 ) {
                        destroy(t)
                    }
                }
            })

            onCollide("enemy", "stopper", (e,s) => {
                e.enterState("idle")
                e.distance = 0
                // console.log(e)
            })

            onCollide("enemy", "basestopper", (e,s) => {
                e.enterState("idle")
                e.distance = 0
                shake(10)
                play("explode")
                destroy(e)
                allItem[0].children.forEach((c,i) => {
                    if (c.name == "base") {
                        if (c.id == s.parent.id) {
                            c.color=RED
                        }
                    }
                })
                addExplode(e.pos, 2, 48, 2)
                if (basehealth > 30) {
                    basehealth -= 35
                    healthBar.width = basehealth*2
                } else {
                    basehealth = 0
                    healthBar.width = basehealth*2
                    wait(0.5, () => {
                        go("lose")
                    })
                }
            })

            onCollide("bullet", "enemy", (b, e) => {
                // console.log(e.color)
                e.color=RED
                destroy(b)
                e.hurt(1)
                play("hit")
                addExplode(b.pos, 1, 24, 1)
                wait(0.2, () => {
                    e.color=WHITE
                })
            })

            on("hurt", "enemy", (e) => {
                shake(1)
            })

            on("death", "enemy", (e) => {
                wait(0.1, () => {
                    destroy(e)
                    shake(2)
                    if (n < 2 ) {
                        budget += 2
                    } else if ( n >= 2 && n < 4) {
                        budget += 3
                    } else if ( n >= 4 ) {
                        budget += 4
                    }
                    
                    label.text=budget
                })
            })

            onCollide("tower", "enemy", (t, e) => {
                destroy(e)
                destroy(t)
                allItem[0].children.filter(d => d.id == t.grid_id)[0].tower = false
                shake(10)
                addExplode(t.pos, 1, 24, 1)
            })

            onCollide("blocker", "enemy", (t, e) => {
                destroy(e)
                destroy(t)
                play("explode")
                allItem[0].children.filter(d => d.id == t.grid_id)[0].tower = false
                shake(10)
                addExplode(t.pos, 1, 24, 1)
                if (n < 2 ) {
                    budget += 2
                } else if ( n >= 2 && n < 4) {
                    budget += 3
                } else if ( n >= 4 ) {
                    budget += 4
                }
                label.text=budget
            })

            onDestroy("enemy", () => {
                d++ // count destroy
                if (d == wave[n]) {
                    d = 0
                    if (n == wave.length-2) {
                        updateW(0)
                        onGame = false
                        wait(1,() => {
                            if (world == "Farmland") {
                                showWaveText("FINAL WAVE", choose(farm_facts))
                            } else if (world == "Mountain") {
                                showWaveText("FINAL WAVE", choose(mountain_facts))
                            } else if (world == "Forest") {
                                showWaveText("FINAL WAVE", choose(forest_facts))
                            }
                            n++
                        })
                    } else if (n < wave.length-1) {
                        updateW(0)
                        onGame = false
                        wait(1,() => {
                            if (world == "Farmland") {
                                showWaveText("WAVE " + (n+2), choose(farm_facts))
                            } else if (world == "Mountain") {
                                showWaveText("WAVE " + (n+2), choose(mountain_facts))
                            } else if (world == "Forest") {
                                showWaveText("WAVE " + (n+2), choose(forest_facts))
                            }
                            n++
                        })
                    } else {
                        updateW(0)
                        onGame = false
                        wait(1, () => {
                            go("finish")
                        })
                    }
                }
            })
            wait(1, () => {
                if (world == "Farmland") {
                    showWaveText("WAVE 1", choose(farm_facts))
                } else if (world == "Mountain") {
                    showWaveText("WAVE 1", choose(mountain_facts))
                } else if (world == "Forest") {
                    showWaveText("WAVE 1", choose(forest_facts))
                }
            })

            onClick("wavebutton", () => {
                wavebox.destroy()
                onGame = true
                let towers = get("tower")
                towers.forEach(t => {
                    shootBullet(t.grid_id, t.pos.x, t.pos.y, t.y_index)
                })
                // startLoop()
                spawnEnemy()
                
            })

        })

        scene("lose", () => {
            onGame = false
            music.paused = true
            add([
                text("YOU LOSE", {font:"Jua"}),
                pos(width()/2, height()/9*4),
                anchor("center"),
                color(BLACK)
            ])

            add([
                rect(250,60, {radius:4}),
                pos(width()/2, height()/9*5),
                anchor("center"),
                outline(2),
                area(),
                "replay"
            ]).add([
                text("PLAY AGAIN", {font:"Nunito", size:20}),
                color(BLACK),
                anchor("center")
            ])

            add([
                rect(250,60, {radius:4}),
                pos(width()/2, height()/9*6),
                anchor("center"),
                outline(2),
                area(),
                "quitbutton"
            ]).add([
                text("MENU", {font:"Nunito", size:20}),
                color(BLACK),
                anchor("center")
            ])

            onClick("replay",() => {
                go("startGame", {world:world})
            })

            onClick("quitbutton",() => {
                isMenu = true
            })
        })

        scene("finish", () => {
            onGame = false
            music.paused = true
            add([
                text("YOU WIN", {font:"Jua"}),
                pos(width()/2, height()/9*4),
                anchor("center"),
                color(BLACK)
            ])
            add([
                rect(250,60, {radius:4}),
                pos(width()/2, height()/9*5),
                anchor("center"),
                outline(2),
                area(),
                "quitbutton"
            ]).add([
                text("MENU", {font:"Nunito", size:20}),
                color(BLACK),
                anchor("center")
            ])

            onClick("quitbutton",() => {
                isMenu = true
            })
        })
    }

    $: if (!isMenu) {
        go("startGame", {world:world})
    }

    function start() {}

    
</script>