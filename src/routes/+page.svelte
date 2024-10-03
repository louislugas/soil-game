<script>
    import kaplay from "kaplay"
    import { browser } from '$app/environment'
    import { onMount } from 'svelte'

    let canvasCollection, canvasArray

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

    function setPos(b,w) {
        return b/2-(3*w)
    }

    $: if (browser) {
        kaplay({
            background: [74, 48, 82],
        })

        const BULLET_SPEED = 400
	    const ENEMY_SPEED = 120
        const OBJ_HEALTH = 2
        
        let budget = 15

        loadSprite("bean", "/images/sprites/bean.png");
        loadSprite("ghosty", "/images/sprites/ghosty.png");
        loadSprite("grass", "/images/sprites/grass.png");

        let grids = []
        let square = 144
        let side = Math.sqrt(square)
        let w = 64
        let yInitPos = 0, xInitPos = 0

        function checkOrientation() {
            if (width() > height()) {
                //landscape
                w = height()/6
                xInitPos = setPos(width(),w)
                yInitPos = w/2
            } else if (height() > width()) {
                //portrait
                w = width()/6
                yInitPos = setPos(height(),w)
                xInitPos = w/2
            }
        }

        checkOrientation()

        onResize(() => {
            checkOrientation()
        })        

        const level = addLevel([
            // Design the level layout with symbols
            "######",
            "######",
            "######",
            "######",
            "######",
            "######",
        ], {
            // The size of each grid
            tileWidth: w,
            tileHeight: w,
            // The position of the top left block
            pos: vec2(xInitPos, yInitPos),
            // Define what each symbol means (in components)
            tiles: {
                "#": () => [
                    sprite("grass"),
                    area(),
                    anchor("center"),
                    scale(w/64),
                    "grid",
                    "ground",
                    "tile",
                    {
                        tower : false,
                        name : "grid"
                    }
                ]
            },
        });

        const allItem = get("*")

        const yPos = [...new Set(allItem[0].children.filter(d=>d.name == "grid").map(d => d.pos.y))]

        const xMax = Math.max(...allItem[0].children.filter(d=>d.name == "grid").map(d => d.pos.x))

            let label = add([
                text(budget),
                pos(width()/2, 50),
                anchor("center")
            ])

            let help = add([
                text("click the tile to add tower that'll shoot the enemy, it cost 5 points", {
                    width:width()/2,
                    size:24
                }),
                pos(width()/2, 100),
                anchor("center")
            ])

        loop(2, () => {
            budget++
            label.text=budget
        })

        onClick("grid", (d) => {
            console.log("grid clicked")
            // console.log(d)
            // console.log(d.tower)
            if (!d.tower) {
                if (budget >= 5) {
                    d.tower = true
                    addSprite(d.id, d.pos.x+setPos(width(),64), d.pos.y+setPos(height(),64))
                    budget-=5
                    label.text = budget
                }
            } else {
                console.log("tower already exist")
            }
        })

        onAdd("tower", (d) => {
            shootBullet(d.grid_id, d.pos.x, d.pos.y)
        })

        function addSprite(id,x,y) {
            add([
                sprite("bean"), 
                pos(x, y),
                anchor("center"),
                area(),
                "tower",
                {grid_id:id}
                ]);
        }

        function removeSprite(id) {

        }

        function shootBullet(id, x,y) {
            let selGrid = allItem[0].children.map(d => d.id).findIndex(d => d == id)
            if (allItem[0].children[selGrid].tower) {
                add([
                    rect(12, 12),
                    area(),
                    pos(x,y),
                    anchor("center"),
                    color(127, 127, 255),
                    outline(4),
                    move(RIGHT, BULLET_SPEED),
                    offscreen({ destroy: true }),
                    // strings here means a tag
                    "bullet",
                ])
                wait(2, () => shootBullet(id,x,y))
            }
        }

        function spawnEnemy() {
            add([
                sprite("ghosty"),
                area(),
                scale(w/64),
                pos(xMax+width()/3+100, yPos[randi(yPos.length)]+setPos(height(),64)),
                health(OBJ_HEALTH),
                anchor("center"),
                "trash",
                "enemy",
                { 
                    speed: rand(ENEMY_SPEED * 0.5, ENEMY_SPEED * 1.5) 
                },
            ])
            wait(5,spawnEnemy)
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
                            rect(4, 4),
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
            t.move(-t.speed, 0)
            if (t.pos.x + t.width < width()/3 ) {
                destroy(t)
            }
        })

        onCollide("bullet", "enemy", (b, e) => {
            destroy(b)
            e.hurt(1)
            addExplode(b.pos, 1, 24, 1)
        })

        on("hurt", "enemy", (e) => {
            shake(1)
        })

        on("death", "enemy", (e) => {
            destroy(e)
            shake(2)
        })

        onCollide("tower", "enemy", (t, e) => {
            destroy(e)
            destroy(t)
            allItem[0].children.filter(d => d.id == t.grid_id)[0].tower = false
            shake(120)
            addExplode(center(), 12, 120, 30)
        })

        spawnEnemy()

    }

    function start() {}

    
</script>