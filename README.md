### Template project for knock-off AliExpress STM32 (CKS32F103C8T6)
## Running
- run `echo "set auto-load safe-path $(pwd)" >> ~/.gdbinit`
- run `./connect` to start openocd
- in another terminal, `cargo run`
- at gdb prompt, type `continue`

## setting up (for another project)
- install `openocd`
- `rustup target install thumbv7m-none-eabi`
- add to cargo.toml:
```toml
[dependencies]
embedded-hal = "0.2.3"
nb = "0.1.2"
cortex-m = "0.6.2"
cortex-m-rt = "0.6.11"
panic-halt = "0.2.0"

[dependencies.stm32f1xx-hal]
version = "0.6.1"
features = ["rt", "stm32f103", "medium"]
```

- put into project_root/.cargo/config:
```
[target.thumbv7m-none-eabi]
runner = 'gdb-multiarch'
rustflags = [
  "-C", "link-arg=-Tlink.x",
]

[build]
target = "thumbv7m-none-eabi"
```
- put into project_root/memory.x:
```
/* Linker script for the STM32F103C8T6 */
MEMORY
{
  FLASH : ORIGIN = 0x08000000, LENGTH = 64K
  RAM : ORIGIN = 0x20000000, LENGTH = 20K
}
```

- put into project_root/.gdbinit:
```
target remote :3333

monitor arm semihosting enable

# # send captured ITM to the file itm.fifo
# # (the microcontroller SWO pin must be connected to the programmer SWO pin)
# # 8000000 must match the core clock frequency
# monitor tpiu config internal itm.fifo uart off 8000000

# # OR: make the microcontroller SWO pin output compatible with UART (8N1)
# # 2000000 is the frequency of the SWO pin
# monitor tpiu config external uart off 8000000 2000000

# # enable ITM port 0
# monitor itm port 0 on

load
step
```

Project should build now (`cargo build` runs without error)

- find openocd's `stm32f1x.cfg` file on system using `find / -name "stm32f1x.cfg"`. Usually in `/usr/share/openocd/scripts/target/stm32f1x.cfg`
- copy this into project root as `cks32f1x.cfg`
- in this file, find the line with `set _CPUTAPID 0x1ba01477` and replace with `set _CPUTAPID 0x2ba01477`
- in project root, run `echo "set auto-load safe-path $(pwd)" >> ~/.gdbinit`
- to connect to MCU, run `openocd -f interface/stlink-v2.cfg -f ./cks32f1x.cfg`
	- stlink filename can be changed based on what link is used
	- can put this in a shell script for convenience
- while openocd is running, `cargo run`. at the gdb prompt type `continue`