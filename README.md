# Transmission (TX)

Transmission is an event driven variant of the Flux front end architecture. For a more academic treatment of the motivation and the principles behind this architecture, read the [whitepaper](./Transmission-TX-A-new-Flux-architecture.pdf). For a more practical guide, continue reading.

# Introduction

There is more than one way to think about *Transmission*. Here we will develop the thinking behind each layer of the architecture peacemeal.

## Principle #1: UI layer is just an empty shell. It needs an "engine" to tell it what to do.

<p align="center"><img src="Transmission-FE1.svg" /></p>

## Principle #2: Every UI state variable must correspond directly to something visible on screen

<p align="center"><img src="Transmission-FE2.svg" /></p>
