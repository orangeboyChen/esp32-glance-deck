# ESP32 Glance Deck Workspace

This repository assembles the Glance Deck projects as Git submodules:

- [Home Assistant integration](home-assistant)
- [Backend control plane](console)
- [ESP32 firmware](firmware)

Clone the workspace and initialize all submodules:

    git clone https://github.com/orangeboyChen/esp32-glance-deck.git
    cd esp32-glance-deck
    git submodule update --init --recursive

Each submodule has its own README, tests, release lifecycle, and GitHub
Actions workflow.
