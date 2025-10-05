node Room {
    has string name;
}

edge Corridor {}

agent Intruder {
    has bool caught = false;
    has bool escaped = false;
}

walker Guard {
    has int patrol_speed = 1;

    can patrol entry {
        log("👮 Guard starts patrol at " + here.name);
        patrol();
    }

    can patrol {
        visit * {
            log("👮 Guard patrolling " + here.name);
            check();
            patrol();
        }
    }

    can check {
        for agent a in here {
            if (typeof a == Intruder and !a.caught and !a.escaped) {
                a.caught = true;
                log("🚨 Intruder caught in " + here.name + "!");
            }
        }
    }
}

walker IntruderMove {
    has int step_count = 0;
    has int max_steps = 10;

    can sneak entry {
        log("🕵️ Intruder begins sneaking from " + here.name);
        sneak();
    }

    can sneak {
        if (step_count >= max_steps) {
            log("⏱ Intruder ran out of time.");
            return;
        }

        step_count += 1;
        if (here.name == "Exit") {
            for agent a in here {
                if (typeof a == Intruder and !a.caught) {
                    a.escaped = true;
                    log("🏃 Intruder ESCAPED successfully!");
                }
            }
            return;
        }

        # Randomly pick a connected room to move into
        if (len(outgoing) > 0) {
            index = rand(0, len(outgoing) - 1);
            target = outgoing[index].sink;
            log("🕵️ Intruder sneaks into " + target.name);
            visit target {
                sneak();
            }
        } else {
            log("😬 Intruder stuck in " + here.name);
        }
    }
}

graph building {
    Room lobby = spawn node Room(name="Lobby");
    Room office = spawn node Room(name="Office");
    Room vault = spawn node Room(name="Vault");
    Room server = spawn node Room(name="Server Room");
    Room exit = spawn node Room(name="Exit");

    lobby - Corridor -> office;
    office - Corridor -> vault;
    office - Corridor -> server;
    server - Corridor -> exit;
    vault - Corridor -> server;
}

with graph building {
    # Spawn intruder
    Intruder bad_guy = spawn agent Intruder at lobby;

    # Spawn intruder walker
    spawn walker IntruderMove at lobby;

    # Spawn multiple guards
    spawn walker Guard at vault;
    spawn walker Guard at server;
}
