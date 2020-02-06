# ffxiv-exp-meter

[Final Fantasy XIV](https://www.finalfantasyxiv.com) [ACT](https://advancedcombattracker.com) overlay for recording [Engineering](https://na.finalfantasyxiv.com/lodestone/playguide/db/search/?q=Engineering+Manual)/[Survival](https://na.finalfantasyxiv.com/lodestone/playguide/db/search/?q=Survival+Manual) Manual usage

## Installation

```url
https://chalkpe.github.io/ffxiv-exp-meter/
```

Support both [OverlayPlugin](https://github.com/ngld/OverlayPlugin) and [ACTWebSocket](https://github.com/ZCube/ACTWebSocket) (requires `BeforeLogLineRead`)

## Usage

### Commands

* `/e expm pause` - temporarily ignore every exp gains
* `/e expm resume` - reverse of `/e expm pause` command
* `/e expm reset` - reset current exp points recoreded
* `/e expm boost <number>` - set boost value (unit: percent)

## Development

### Local development

```sh
yarn && yarn serve
```

### Deploy to GitHub Pages

```sh
yarn build && yarn deploy
```

## License

[MIT License](LICENSE)
