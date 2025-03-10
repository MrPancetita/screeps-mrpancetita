# Screeps Tutorial - Lesson 1: Game UI and basic scripting

Your spawn creates new units called "creeps" by its method spawnCreep. Usage of this method is described in the documentation. Each creep has a name and certain body parts that give it various skills.

You can address your spawn by its name the following way: `Game.spawns['Spawn1']`.

Create a worker creep with the body array `[WORK,CARRY,MOVE]` and name `Harvester1` (the name is important for the tutorial!). You can type the code in the console yourself or copy & paste the hint below.

[Your-colony](https://docs.screeps.com/introduction.html#Your-colony)
[Creeps](https://docs.screeps.com/creeps.html)
[Game-object](https://docs.screeps.com/global-objects.html#Game-object)
[StructureSpawn](https://docs.screeps.com/api/#StructureSpawn)

```js
Game.spawns['Spawn1'].spawnCreep( [WORK, CARRY, MOVE], 'Harvester1' );
```

It is time to put the creep to work! This yellow square is an energy source — a valuable game resource. It can be harvested by creeps with one or more `WORK` body parts and transported to the spawn by creeps with `CARRY` parts.

In the script tab. Here you can write scripts that will run on a permanent basis, each game tick in a loop. It allows writing constantly working programs to control behaviour of your creeps which will work even while you are offline (in the real game only, not the Simulation Room mode).

To commit a script to the game so it can run, use this button or Ctrl+Enter.

The code for each Tutorial section is created in its own branch. You can view code from these branches for further use in your scripts.

Documentation:
[Scripting basics](https://docs.screeps.com/scripting-basics.html)

To send a creep to harvest energy, you need to use the methods described in the documentation section below. Commands will be passed each game tick. The harvest method requires that the energy source is adjacent to the creep.

You give orders to a creep by its name this way: `Game.creeps['Harvester1']`. Use the `FIND_SOURCES` constant as an argument to the `Room.find` method.

Send your creep to harvest energy by typing code in the "Script" tab.
Documentation:

[Game.creeps](https://docs.screeps.com/api/#Game.creeps)
[RoomObject](https://docs.screeps.com/api/#RoomObject)
[Room.find](https://docs.screeps.com/api/#Room.find)
[Creep.moveTo](https://docs.screeps.com/api/#Creep.moveTo)
[Creep.harvest](https://docs.screeps.com/api/#Creep.harvest)

```js
module.exports.loop = function () {
    var creep = Game.creeps['Harvester1'];
    var sources = creep.room.find(FIND_SOURCES);
    if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
        creep.moveTo(sources[0]);
    }
}
```

A bubbling yellow spot inside the creep means that it has started collecting energy from the source.

To make the creep transfer energy back to the spawn, you need to use the method `Creep.transfer`. However, remember that it should be done when the creep is next to the spawn, so the creep needs to walk back.

If you modify the code by adding the check `.store.getFreeCapacity() > 0` to the creep, it will be able to go back and forth on its own, giving energy to the spawn and returning to the source.

Extend the creep program so that it can transfer harvested energy to the spawn and return back to work.

Documentation:
[Creep.transfer](https://docs.screeps.com/api/#Creep.transfer)
[Creep.store](https://docs.screeps.com/api/#Creep.store)

```js
module.exports.loop = function () {
    var creep = Game.creeps['Harvester1'];

    if(creep.store.getFreeCapacity() > 0) {
        var sources = creep.room.find(FIND_SOURCES);
        if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
            creep.moveTo(sources[0]);
        }
    }
    else {
        if( creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE ) {
            creep.moveTo(Game.spawns['Spawn1']);
        }
    }
}
```

Great! This creep will now work as a harvester until it dies. Remember that almost any creep has a life cycle of 1500 game ticks, then it "ages" and dies (this behavior is disabled in the Tutorial).

Let's create another worker creep to help the first one. It will cost another 200 energy units, so you may need to wait until your harvester collects enough energy. The `spawnCreep` method will return an error code `ERR_NOT_ENOUGH_ENERGY` (-6) until then.

Remember: to execute code once just type it in the "Console" tab.

Spawn a second creep with the body [WORK,CARRY,MOVE] and name Harvester2.
Documentation:
[StructureSpawn.spawnCreep](https://docs.screeps.com/api/#StructureSpawn.spawnCreep)

```js
Game.spawns['Spawn1'].spawnCreep( [WORK, CARRY, MOVE], 'Harvester2' );
```

The second creep is ready, but it won't move until we include it into the program.

To set the behavior of both creeps we could just duplicate the entire script for the second one, but it's much better to use the for loop against all the screeps in Game.creeps.

Expand your program to both the creeps.
Documentation:
[JavaScript for loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in)

 ```js
 module.exports.loop = function () {
    for(var name in Game.creeps) {
        var creep = Game.creeps[name];

        if(creep.store.getFreeCapacity() > 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0]);
            }
        }
        else {
            if(creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
                creep.moveTo(Game.spawns['Spawn1']);
            }
        }
    }
}
```

Now let's improve our code by taking the workers' behavior out into a separate module. Create a module called `role.harvester` with the help of the Modules section on the left of the script editor and define a `run` function inside the `module.exports` object, containing the creep behavior.

[Organizing code in modules](https://docs.screeps.com/modules.html)

```js
var roleHarvester = {

    /** @param {Creep} creep **/
    run: function(creep) {
        if(creep.store.getFreeCapacity() > 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0]);
            }
        }
        else {
            if(creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
                creep.moveTo(Game.spawns['Spawn1']);
            }
        }
    }
};

module.exports = roleHarvester;
```

Now you can rewrite the main module code, leaving only the loop and a call to your new module by the method `require('role.harvester')`.

Include the `role.harvester` module in the main module.

```js
var roleHarvester = require('role.harvester');

module.exports.loop = function () {

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        roleHarvester.run(creep);
    }
}
```
