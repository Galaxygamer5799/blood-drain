# blood-drain
import { system, MolangVariableMap } from '@minecraft/server'
import { setScore, getScore, delayedFunc, playSound } from "../../util.js";

const command = {
    name: 'Blood Drain',
    description: 'Drains the blood from players for a few seconds, however, they can still take damage. Are they about to mlg? You can stop that.',
    style: 'water',
    sub_bending_required: 'blood',
    unlockable: 0,
    unlockable_for_avatar: 0,
    off_tier_required: 2,
    cooldown: 'fast',
    execute(player) {
        // Set cooldown so they can't spam
        setScore(player, "cooldown", 10);

        // Check if they have water
        player.playAnimation("animation.water.blast");
        delayedFunc(player, (bloodDrain) => {
            playSound(player, 'mob.turtle.swim', 0.9, player.location, 1);
            player.addEffect("speed", 15, { amplifier: 2, showParticles: false });
            player.runCommand(`inputpermission set @a[r=10,name=!"${player.name}"] movement disabled`);
            player.runCommand(`inputpermission set @a[r=10,name=!"${player.name}"] camera disabled`);
    
            let currentTick = 0;
            const sched_ID = system.runInterval(function tick() {
                // In case of errors
                currentTick++;
                if (currentTick > 350) return system.clearRun(sched_ID);
                player.runCommand(`execute as @e[type=!player,r=10] at @s run tp @s @s`);
                const entities = [...player.dimension.getEntities({ location: player.location, maxDistance: 16 })];
                entities.forEach(entity => {
                    player.dimension.spawnParticle("a:redstone_ore_dust_particle", entity.location, new MolangVariableMap());
                });
                if (currentTick > 20) {
                    player.runCommand(`inputpermission set @a[r=10,name=!"${player.name}"] movement enabled`);
                    player.runCommand(`inputpermission set @a[r=10,name=!"${player.name}"] camera enabled`);
                    return system.clearRun(sched_ID);
                }
            }, 1)
        }, 10)
    }
}

export default command
