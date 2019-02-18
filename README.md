# Dockerized dbus playback + mqtt service

Use this docker to run an mqtt broker that serves recorded dbus data. Main use case would be to run this instead of having a real Venus GX device or VenusOs running in a raspberry pi, making local development work much easier but also more realistic. The container can run different configurations (different sets of devices connected to it) of the VenusGX by running different sets of recordings with a simple commandline api.

## Usage

- Install Docker
- Create container with `./build.sh`
- Run the container as an interactive shell with `./run.sh`
  - run `./start_services.sh` within the container to start the mqtt and other services.
  - run `./simulate.sh <simulation>` to start playback.
- Run the container in the background with a simulation with `./run.sh -s <simulation_name>`
  - to kill the container (and to remove it because of the `--rm` option) use `docker kill <container id>`
  - get the container id from the output of the run script or with `docker ps`
- For more info run `./run.sh -h`

### Working inside the container

You can see what data is available in the mqtt by using `mosquitto_sub -t N/#` or use an mqtt spy application. To change values manually use `mosquitto_pub`, but these values are likely to be overridden by an active recording quite quickly.

## Modifying recordings

Recordings can be modified with the following steps:

1. Get the recording desired as a tsv file: `./get_recording_tsv.sh <simulation>`. Simulation must be a simulation in ./simulations/X/Y.dat
2. Edit acquired tsv
3. Recompile the simulation and replace local simulation file with `./recompile_and_replace_simulation.sh <edited_simulation.tsv> <simulation`. Simulation must be the same simulation file as in step 1

## Systems (simulations)

Battery selection should always be on auto; unless specified differently.

### A) Absolut Navetta 68 installation

2 x Skylla-i chargers
2 x VE.Bus Phoenix Inverter 24/3000 (230V and 120V for USA)
1 x BMV-700

And a few more devices, but they won't be connected to Venus:

1 x Phoenix charger 24/25 (for engines)
1 x BlueSmart 12/30 IP22 (for generators) Which electrical parameters can be monitored with these machines? (device by device) Is it possible to set alarms or change device functions via the interface with Garmin?"
AC Input 1 & 2 settings are (probably) to be configured as not available; we'll find out once we start working on the gui-overview for this.

### B) Single BMV-700

Settings:

- DC system enabled

Show in html5app:

- battery box
- dc loads

### C) Single BMV-702

BMV configured to measure starter battery voltage.

Settings:

- DC-system enabled

Show in html5app:

- main battery box
- simple battery box with only voltage (the starter battery)
- dc loads

### D) Multi + BMV - Off-grid with generator

VE.Bus:

- CurrentlimitIsAdjustable = false.
- Mode is adjustable = true.

Settings:

- AC input type 1 = Generator
- AC input type 2 = Not available
- DC system disabled

### E) Multi + BMV - Boat without generator

VE.Bus:

- CurrentlimitIsAdjustable = true.
- Mode is adjustable = true.

Settings:

- AC input type 1 = Shore
- AC input type 2 = Not available
- DC system enabled

### F) Quattro + BMV - boat with generator - single phase

VE.Bus:

- AcIn/0/CurrentlimitIsAdjustable-ac-input = false.
- AcIn/1/CurrentlimitIsAdjustable-ac-input = true.
- Mode is adjustable = true.

Settings:

- AC input type 1 = Generator
- AC input type 2 = Shore
- DC system enabled

### G) Charger + BMV - simple boat

Settings:

- AC input type 1 = shore
- AC input type 2 = not available
- DC system enabled

### H) VE.Direct Inverter + BMV - typical simple vehicle - only charged from alternator

Settings:

- AC input types both on not available
- DC system enabled

### I) Quattro without BMV - Hybrid generator - single phase

VE.Bus:

- AcIn/0/CurrentlimitIsAdjustable-ac-input = false.
- AcIn/1/CurrentlimitIsAdjustable-ac-input = true.
- Mode is adjustable = true.

Settings:

- AC Input type 1 = generator
- AC input type 2 = grid
- DC system disabled

### J) 4 x BMV-700

Settings:

- DC system enabled

### K) Quattro without BMV - Hybrid generator - three phase

VE.Bus:

- AcIn/0/CurrentlimitIsAdjustable-ac-input = false.
- AcIn/1/CurrentlimitIsAdjustable-ac-input = true.
- Mode is adjustable = true.

Settings:

- AC Input type 1 = generator
- AC input type 2 = grid
- DC system disabled

### L) Quattro + BMV - boat with generator - three phase

VE.Bus:

- AcIn/0/CurrentlimitIsAdjustable-ac-input = false.
- AcIn/1/CurrentlimitIsAdjustable-ac-input = true.
- Mode is adjustable = true.

Settings:

- AC input type 1 = Generator
- AC input type 2 = Shore
- DC system enabled

### M) Multi with a VE.Bus BMS

Note https://github.com/victronenergy/venus-private/issues/86. The demo mode is now
not how reality is. And how reality will be is unknown as of yet.

VE.Bus:

- AcIn/0/CurrentlimitIsAdjustable-ac-input = false.
- AcIn/1/CurrentlimitIsAdjustable-ac-input = false.
- Mode is adjustable = false.

Settings:

- AC input type 1 = Generator
- AC input type 2 = Shore
- DC system disabled

## Building dbus-spy

A binary copy of dbus-spy is already included in this repo, but should you need
to rebuild it, these are the steps:

    git clone git@github.com:victronenergy/dbus-spy.git
	cd dbus-spy
	git submodule update --init
	cd ..
	docker build -f dockerfile.dbus-spy . -t dbus-spy

Then copy dbus-spy out of a throwaway container by first starting a container:

    docker run -it --rm dbus-spy

Then copy dbus-spy to bin:

	docker cp <container>:/usr/local/bin/dbus-spy bin/dbus-spy
