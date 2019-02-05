# Rendering Comet Avatars

The CometVerse website provides multiple endpoints to render Comets with their items equipped. These URLs can be used within HTML `<img>` tags.

### ID Parameters
The `<id>` URL parameter in the below examples can either be the Comet Token ID (`123`) or an address (`0x...`).

## Full Comet:
URL: `https://www.cometverse.com/comet/<id|address>.svg`

Usage: `<img src="https://www.cometverse.com/comet/<id|address>.svg" alt="Mini Comet" height="256" width="256">`
<img src="https://www.cometverse.com/comet/3.svg" alt="Mini Comet" height="256" width="256">

## Mini Comet:
URL: `https://www.cometverse.com/comet/<id|address>/mini.svg`

Usage: `<img src="https://www.cometverse.com/comet/<id|address>/mini.svg" alt="Mini Comet" height="256" width="256">`
<img src="https://www.cometverse.com/comet/3/mini.svg" alt="Mini Comet" height="256" width="256">
