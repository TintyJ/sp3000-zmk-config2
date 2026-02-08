# Register the board with Zephyr/ZMK
board_runner_args(nrfutil "--dev-type" "usb")

# Include the devicetree and Kconfig for this board
zephyr_library()
zephyr_library_sources(sp3000_keypad.dts)
