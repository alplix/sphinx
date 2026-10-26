# Sphinx

Sphinx is the [BitBOINC](https://bitboinc.athena.org.tr/) project's attempt at a **Certicom ECC challenge**: the elliptic-curve discrete logarithm problem on the 131-bit prime-field curve **ECCp-131**, using distributed Pollard-kangaroo walks.

> **This repository contains documentation only. It does not contain source code, work-unit parameters, search state or server code.** See [What is not published, and why](#what-is-not-published-and-why).

## The problem

Certicom published a series of elliptic-curve discrete-logarithm challenges of growing size so that the real cost of attacking such curves could be measured. ECCp-131 is a curve over a prime field of about 131 bits: given a point `Q = k·P`, find `k`. The generic attack needs on the order of the square root of the group order, a very large amount of computation, so this is a long-term effort and a benchmark of what volunteer computing can reach. Whether it can be solved with the compute available to the project is an open question.

## The method

- **Kangaroo walks with distinguished points**, as in Keraunos, but on the Certicom curve: its own field arithmetic (the field elements need three 64-bit words) and its own group order. The walk distances have to be reduced modulo **that** order, not modulo the order of Bitcoin's secp256k1 curve; mixing them up would silently corrupt every result, so the two code paths are kept separate.
- Tame and wild kangaroos jump through a table of pseudo-random steps; a point with a fixed number of leading zero bits is a **distinguished point (DP)** and is reported with its travelled distance. A tame/wild collision yields `k`.

## How it runs on BOINC

- Volunteers' computers run the walks (CPU on many platforms, GPU through CUDA, OpenCL and Metal), with checkpoints so a task can resume.
- DPs are sent to the project server as BOINC *trickle* messages while the task runs. A server-side daemon stores them and looks for collisions.
- Credit is a flat amount per task.

## Correctness and safety

- **Self-tests** on the volunteer's machine check the curve arithmetic before the walks start. A device that miscomputes refuses to run, so wrong points are not sent.
- **Server-side verification:** a reported collision is turned into a candidate `k`; the server checks `k·P` against the challenge point before it counts the challenge as solved.

## What happens if the key is found

The solution would be handled like the project's other keys: encrypted immediately with an RSA-4096 (OAEP) public key, only the ciphertext stored, the private key kept offline by the project owner and never on the server.

## What volunteers' computers do and do not do

They compute walks and send DPs. They do not touch any wallet or file of the user, and the project stores no personal data about the search itself. See the project's privacy page.

## What is not published, and why

The source code of the optimized implementation, the tuning parameters and the accumulated DPs are deliberately **not published**: the DPs are the work of the volunteers, and a fast implementation is an advantage in a challenge that rewards the first solver. The design above is published so that it can be reviewed without the solver.

## License

The text of this repository is released under the GPL-3.0 (see `LICENSE`).
