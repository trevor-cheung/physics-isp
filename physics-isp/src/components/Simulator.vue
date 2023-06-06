<script setup>
import Matter from 'matter-js'

var render;
var engine;
var runner;
var cue, balls1, balls2, balls3, balls4, balls5, composite, world, rock, anchor, anchorBall, moving, elastic;
var Engine = Matter.Engine,
        Render = Matter.Render,
        Runner = Matter.Runner,
        Composites = Matter.Composites,
        Common = Matter.Common,
        Constraint = Matter.Constraint,
        MouseConstraint = Matter.MouseConstraint,
        Mouse = Matter.Mouse,
        Composite = Matter.Composite,
        Events = Matter.Events,
        Bodies = Matter.Bodies,
        Body = Matter.Body;
var forceX, forceY = 0;
var posX = 300;
var posY = 300;
var position, velocity, aVelocity;

function init() {
    // create engine
    engine = Engine.create(),
        world = engine.world;
        engine.gravity.scale = 0;

    // create renderer
    render = Render.create({
        element: document.body,
        engine: engine,
        options: {
            width: 1200,
            height: 600,
            showAngleIndicator: true,
        }
    });

    Render.run(render);

    // create runner
    runner = Runner.create();
    Runner.run(runner, engine);

    // cue ball
    cue = Bodies.circle(300, 300, 13.5, { restitution: 0.9, friction: 0.2, mass: 1, isStatic: false })

    // other balls
    balls1 = Composites.stack(900, 232.5, 1, 5, 0, 0, function(x, y) {
        return Bodies.circle(x, y, 13.5, {
            render: {
                fillStyle: 'cornflowerblue',
                strokeStyle: 'black'
            }, restitution: 0.9, friction: 0.2, mass: 1
        })
    })
    balls2 = Composites.stack(877, 246, 1, 4, 0, 0, function(x, y) {
        return Bodies.circle(x, y, 13.5, {
            render: {
                fillStyle: 'cornflowerblue',
                strokeStyle: 'black'
            }, restitution: 0.9, friction: 0.2, mass: 1
        })
    })
    balls3 = Composites.stack(854, 259.5, 1, 3, 0, 0, function(x, y) {
        return Bodies.circle(x, y, 13.5, {
            render: {
                fillStyle: 'cornflowerblue',
                strokeStyle: 'black'
            }, restitution: 0.9, friction: 0.2, mass: 1
        })
    })
    balls4 = Composites.stack(831, 273, 1, 2, 0, 0, function(x, y) {
        return Bodies.circle(x, y, 13.5, {
            render: {
                fillStyle: 'cornflowerblue',
                strokeStyle: 'black'
            }, restitution: 0.9, friction: 0.2, mass: 1
        })
    })
    balls5 = Composites.stack(808, 286.5, 1, 1, 0, 0, function(x, y) {
        return Bodies.circle(x, y, 13.5, {
            render: {
                fillStyle: 'cornflowerblue',
                strokeStyle: 'black'
            }, restitution: 0.9, friction: 0.2, mass: 1
        })
    })

    // cue ball sling shot
    rock = Bodies.circle((posX - 50), posY, 13.5, { restitution: 0.9, friction: 0.2, mass: 1, isStatic: false, render: { fillStyle: 'black', strokeStyle: 'black' } }),
        anchor = { x: posX, y: posY },
        anchorBall = Bodies.circle(posX,posY,13.5, { restitution: 0.9, friction: 0.2, mass: 1, isStatic: false }),
        moving = false,
        elastic = Constraint.create({ 
            pointA: anchor, 
            bodyB: rock, 
            length: 50,
            damping: 0.01,
            stiffness: 0.05
        });
    console.log(rock);

    Events.on(engine, 'afterUpdate', function() {
        if (mouseConstraint.mouse.button === -1 && (rock.position.x > anchorBall.position.x+1 || rock.position.y < anchorBall.position.y-1)) {
            // Limit maximum speed of current rock.
            if (Body.getSpeed(rock) > 45) {
                Body.setSpeed(rock, 45);
            }

            // Release current rock
            Composite.remove(world, [elastic, anchorBall]);
            moving = true;
        }
        document.querySelector('#position').innerHTML = 'Position: '+rock.position.x.toFixed(2)+', '+rock.position.y.toFixed(2);
        document.querySelector('#velocity').innerHTML = 'Velocity: '+rock.velocity.x.toFixed(2)+', '+rock.velocity.y.toFixed(2);
        document.querySelector('#aVelocity').innerHTML = 'Angular Velocity: '+rock.angularVelocity.toFixed(2);
        document.querySelector('#speed').innerHTML = 'Speed: '+rock.speed.toFixed(2);
        
        //attempt at moving slingshot to current location
        if (moving == true && rock.speed < 0.01) {
            anchorBall = Bodies.circle(rock.position.x,rock.position.y,13.5, { restitution: 0.9, friction: 0.2, mass: 1, isStatic: false });
            anchor = { x: rock.position.x, y: rock.position.y }
            //rock = Bodies.circle(rock.position.x-50, rock.position.y, 13.5, { restitution: 0.9, friction: 0.2, mass: 1, isStatic: false })
            Body.translate(rock, { x: -50, y: 0 })
            elastic = Constraint.create({ 
                pointA: anchor, 
                bodyB: rock, 
                length: 50,
                damping: 0.01,
                stiffness: 0.05
            });
            moving = false;
            Composite.add(world, [elastic, anchorBall])
            //Composite.remove(world, rockMoving)
        }
    })

    // adding everything into composite
    composite = Composite.add(world, [
        // walls
        Bodies.rectangle(600, 0, 1080, 50, { isStatic: true, restitution: 0.9 }),
        Bodies.rectangle(600, 600, 1080, 50, { isStatic: true, restitution: 0.9 }),
        Bodies.rectangle(1200, 300, 50, 480, { isStatic: true, restitution: 0.9 }),
        Bodies.rectangle(0, 300, 50, 480, { isStatic: true, restitution: 0.9 }),
        Bodies.rectangle(0, 565, 1, 70.7, { isStatic: true, restitution: 0.9, angle: Math.PI/4}),
        Bodies.rectangle(35, 600, 1, 70.7, { isStatic: true, restitution: 0.9, angle: Math.PI/4}),
        Bodies.rectangle(0, 35, 1, 70.7, { isStatic: true, restitution: 0.9, angle: 7*Math.PI/4}),
        Bodies.rectangle(35, 0, 1, 70.7, { isStatic: true, restitution: 0.9, angle: 7*Math.PI/4}),
        Bodies.rectangle(1200, 35, 1, 70.7, { isStatic: true, restitution: 0.9, angle: Math.PI/4}),
        Bodies.rectangle(1165, 0, 1, 70.7, { isStatic: true, restitution: 0.9, angle: Math.PI/4}),
        Bodies.rectangle(1200, 565, 1, 70.7, { isStatic: true, restitution: 0.9, angle: 7*Math.PI/4}),
        Bodies.rectangle(1165, 600, 1, 70.7, { isStatic: true, restitution: 0.9, angle: 7*Math.PI/4}),
        balls1, balls2, balls3, balls4, balls5, rock, elastic, anchorBall
    ]);

    // add mouse control
    var mouse = Mouse.create(render.canvas),
        mouseConstraint = MouseConstraint.create(engine, {
            mouse: mouse,
            constraint: {
                stiffness: 0.2,
                render: {
                    visible: true
                }
            }
        });


    Composite.add(world, mouseConstraint);

    // keep the mouse in sync with rendering
    render.mouse = mouse;

    // fit the render viewport to the scene
    Render.lookAt(render, {
        min: { x: 0, y: 0 },
        max: { x: 1200, y: 600 }
    });    
}

    // cue ball functions
    function applyForce() {
        Composite.remove(world, [elastic, anchorBall]);
        console.log(rock);
        Body.applyForce(rock, rock.position, {x: forceX, y: forceY});
    }

    function setForceX() {
        forceX = parseFloat(document.getElementById('force').value);
    }

    function setForceY() {
        forceY = parseFloat(document.getElementById('forceY').value);
    }

    function setPosX() {
        posX = parseFloat(document.getElementById('posX').value);
    }

    function setPosY() {
        posY = parseFloat(document.getElementById('posY').value);
    }

    function getVelocity() {
        return Body.getVelocity(rock);
    }

    function createTextNode(text) {
        let newCell = document.createElement('p');
        newCell.appendChild(document.createTextNode(text));
        return newCell;
    }

    // reset function
    function reset() {
        Composite.clear(composite)
            Engine.clear(engine);
            Render.stop(render);
            Runner.stop(runner);
            render.canvas.remove();
            render.canvas = null;
            render.context = null;
            render.textures = {};
            console.log('reset clicked');
            render, engine, runner, cue, balls1, balls2, balls3, balls4, balls5, composite, world = null;
            init();
    }

    // initiai init
    init();
</script>

<template>
    <p>Set Force</p>
    <input type="text" id="force" class="input" @input="setForceX" placeholder="X Force (between 0-1)">
    <input type="text" id="forceY" class="input" @input="setForceY" placeholder="Y Force (between 0-1)">
    <button class="button" @click="applyForce()">test</button>
    <button class="button" id="reset" @click="reset()">Reset</button>
    <p>Starting Position (default 300,300 - press reset to apply changes)</p>
    <input type="text" id="posX" class="input" @input="setPosX" placeholder="X Pos">
    <input type="text" id="posY" class="input" @input="setPosY" placeholder="Y Pos"> 
    <div id="position"></div>
    <div id="velocity"></div>
    <div id="aVelocity"></div>
    <div id="speed"></div>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
