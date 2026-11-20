# frame-gride-online

English | [日本語](README.ja.md)

**Unofficial fan project.** Notes, tools and a stand-alone server for the online 2-player mode ("NET GAME") of the Dreamcast game *FRAME GRIDE* (FromSoftware, 1999, `T34201M`) on the [Flycast](https://github.com/flyinghead/flycast) emulator, with an **unmodified disc**.

> **Update 2026-11-20: FRAME GRIDE is now online on Flycast's DCNet.** Flycast's author added the game to DCNet ([discussion #2621](https://github.com/flyinghead/flycast/discussions/2621), commit [`460a821`](https://github.com/flyinghead/flycast/commit/460a821)). The change is in Flycast's master branch and **not in the 2.7 release**, so use a recent build from the [Flycast builds page](https://flyinghead.github.io/flycast-builds/). The server in this repository is no longer needed to play; it stays here as a reference and as a fallback.

Nothing in the game or in Flycast is patched. No game data is included or distributed.

> This project is not affiliated with or endorsed by FromSoftware, SEGA or the Flycast authors.

## Play online with DCNet

1. Get your own copy of *FRAME GRIDE* (Japan, `T34201M`) as a disc image.
2. Install a recent Flycast build (master) from the [builds page](https://flyinghead.github.io/flycast-builds/). The 2.7 release does not know this game yet.
3. Turn on DCNet in the Network tab of Flycast's settings. The settings (including the emulated modem's ISP entry) are explained on the [Flycast wiki, DCNet page](https://github.com/TheArcadeStriker/flycast-wiki/wiki/DCNet).
4. Start the game and follow [In the game](#in-the-game-net-game) below.

Path delay counts directly against the game's limits (see [Status](#status)): the shorter the path to the DCNet server, the better it plays.

## In the game (NET GAME)

1. Title → **NET GAME** → enter a name → dial (A) → server (A) → nickname (A). You are in the lobby.
2. Player 1: START → create a team (A, then A, A, START). Player 2: START → down → A → join (A).
3. Player 1 in the team room: START → PLAY (A). Both consoles enter the battle.

These steps were worked out with this project's own server. They should be the same on DCNet.

## Status

| What | State |
|---|---|
| DCNet | Added by the Flycast author (Flycast master, 2026-11-20). He showed the lobby and a finished match in [discussion #2621](https://github.com/flyinghead/flycast/discussions/2621). **Not yet tested by this project.** |
| Own server: dial-up, lobby, team creation and join, PLAY | Works |
| Own server: battle to the end, rematch, back to the team room | Works (2 Flycast windows on one PC, automated checks) |
| Own server: tolerance on one PC (emulated) | Stable up to 50 ms one-way delay, 100 ms marginal, 200 ms fails; jitter up to 40 ms and 5 % loss pass |
| Own server: two PCs on one LAN (Windows + macOS) | Both reached the same lobby and stayed connected. A full match across two PCs has not been played. |
| Own server: over the internet | Not tested |
| Unlock-all save tool | Works with the own server: checked in the story mode and in a NET GAME battle (the other player sees the unlocked mech). **Not yet checked on DCNet.** |

The network battle has about 10 frames (~170 ms) more input lag than an offline battle, even with no added delay. See [docs/MEASUREMENTS.md](docs/MEASUREMENTS.md).

## Unlocking everything in your save

The card must already hold a save: play the story mode once, enter a name, save. Then, with Flycast closed:

```
uv run python unlock_all.py T34201M_vmu_save_A1.bin unlocked/T34201M_vmu_save_A1.bin
uv run python vmu_tool.py unlocked/T34201M_vmu_save_A1.bin
```

Battle records are left as they are unless you ask: `--wins N` and `--losses N` (0-9999 each) set the wins and losses of the 1P, VS, NET and TOTAL records, and the number of matches becomes wins + losses (at most 9999). `--max-records` is a shortcut for 9999 matches, 0 losses, 9999 wins. The record with losses has been checked in the file, not yet on the game screen.

The original card is never overwritten; keep a copy anyway. Put the new card in Flycast's data folder. To use the unlocked mech in NET GAME: in the team room, START → 騎兵設定 → 記 録 → ロード → pick the save → B → 設定終了. Details: [docs/SAVE_FORMAT.md](docs/SAVE_FORMAT.md).

On a public server other players see your mech and your records, so please use the unlock and record options with that in mind.

## Run your own server (optional)

The Python code here (standard library only) does the job of the old game servers on its own. It is useful for LAN play, for testing and as a reference for how the game talks.

- `modem_bridge.py` + `ppp_engine.py`: a PPP server for Flycast's DCNet modem connection. It answers the console's dial-up, assigns addresses and routes IP packets between the consoles.
- `lobby_probe.py`: a re-implementation of the game's lobby (UDP 9500/9501): nicknames, lobby list, teams, PLAY.
- `unlock_all.py` + `vmu_tool.py`: the save tools described above.

You need Python 3 (tested with 3.14). [uv](https://docs.astral.sh/uv/) is used in the examples; plain `python` works too.

### One PC, two Flycast windows

1. Start the lobby and the bridge (two terminals):

   ```
   uv run python lobby_probe.py
   uv run python modem_bridge.py --mode ppp --vj asymmetric
   ```

2. In each Flycast, set in `emu.cfg` (section `[network]`):

   ```
   DCNet = yes
   DCNetServer = 127.0.0.1
   ```

   For two windows on one PC, give each instance its own data folder, `LocalPort` and `ISPUsername` (for example `flycast_p1` / `flycast_p2`).

3. Play as described in [In the game](#in-the-game-net-game).

### Two PCs

Run `lobby_probe.py` and `modem_bridge.py` on one PC (the host), with the bridge listening on all interfaces:

```
uv run python modem_bridge.py --mode ppp --vj asymmetric --host 0.0.0.0
```

The other player sets `DCNetServer` to the host's address. Only **TCP 7654** on the host has to be reachable (allow it in the host's firewall). On the internet this means a port forward, or a VPN such as Tailscale. The lobby stays on the host: the bridge forwards the console's lobby packets to it locally.

### Bridge options for testing

`modem_bridge.py` can add delay, jitter and loss between the consoles to rehearse a longer path on one PC: `--p2p-delay-ms`, `--p2p-jitter-ms`, `--p2p-loss-pct`, `--p2p-seed`. Both servers write JSON-lines logs next to the scripts (`modem-capture.jsonl` or the `--log` path, `lobby-probe.jsonl`).

## Tests

```
uv run python -m unittest discover
```

Two tests read a real memory card if you point `FG_SAVE_IMAGE` at one; otherwise they are skipped.

## Documents

- [docs/PROTOCOL.md](docs/PROTOCOL.md): what the bridge and the lobby speak.
- [docs/HANDSHAKE.md](docs/HANDSHAKE.md): the exact PPP handshake the game needs (LCP, PAP, IPCP), byte by byte, and why every frame needs its own leading `0x7E`.
- [docs/MEASUREMENTS.md](docs/MEASUREMENTS.md): delay, jitter, loss and input-lag results.
- [docs/SAVE_FORMAT.md](docs/SAVE_FORMAT.md): the save file layout and the unlock tool.

## License

MIT, see [LICENSE](LICENSE). The license covers this project's code and documents only, not the game or Flycast.
