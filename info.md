## App configuration

```yaml
pixelit:
  module: pixelit
  class: pixelit
  ip: 192.168.178.24
  path: "/config/pixelit"
  entitiy_id: sensor.pixelit
```
key | optional | type | default | description
-- | -- | -- | -- | --
`ip` | False | string | | The ip of the pixel controller (ESP)
`path` | False | string | | The path to the json templates, this has to be outside of the appdaemon/apps/pixelit folder!!
`entity_id` | True | string | `sensor.pixelit`| The entity_id of the PixelIt sensor.
