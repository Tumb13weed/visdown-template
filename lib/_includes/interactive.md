# Using Vega

Make complex interactive visualisations using vega (airports example)

```vis
width: 600
height: 400
padding:
  top: 25
  left: 0
  right: 0
  bottom: 0
data:
- name: states
  url: data/us-10m.json
  format:
    type: topojson
    feature: states
  transform:
  - type: geopath
    projection: albersUsa
    scale: 600
    translate:
    - 300
    - 200
- name: traffic
  url: data/flights-airport.csv
  format:
    type: csv
    parse: auto
  transform:
  - type: aggregate
    groupby:
    - origin
    summarize:
    - field: count
      ops:
      - sum
      as:
      - flights
- name: airports
  url: data/airports.csv
  format:
    type: csv
    parse: auto
  transform:
  - type: lookup
    'on': traffic
    onKey: origin
    keys:
    - iata
    as:
    - traffic
  - type: filter
    test: datum.traffic != null
  - type: geo
    projection: albersUsa
    scale: 600
    translate:
    - 300
    - 200
    lon: longitude
    lat: latitude
  - type: filter
    test: datum.layout_x != null && datum.layout_y != null
  - type: sort
    by: "-traffic.flights"
  - type: voronoi
    x: layout_x
    y: layout_y
- name: routes
  url: data/flights-airport.csv
  format:
    type: csv
    parse: auto
  transform:
  - type: filter
    test: hover && hover.iata == datum.origin
  - type: lookup
    'on': airports
    onKey: iata
    keys:
    - origin
    - destination
    as:
    - _source
    - _target
  - type: filter
    test: datum._source && datum._target
  - type: linkpath
scales:
- name: size
  type: linear
  domain:
    data: traffic
    field: flights
  range:
  - 16
  - 1000
signals:
- name: hover
  init:
  streams:
  - type: "@cell:mouseover"
    expr: datum
  - type: "@cell:mouseout"
    expr: 'null'
- name: title
  init: U.S. Airports, 2008
  streams:
  - type: hover
    expr: 'hover ? hover.name + '' ('' + hover.iata + '')'' : ''U.S. Airports, 2008'''
- name: cell_stroke
  init:
  streams:
  - type: dblclick
    expr: 'cell_stroke ? null : ''brown'''
marks:
- type: path
  from:
    data: states
  properties:
    enter:
      fill:
        value: "#dedede"
      stroke:
        value: white
    update:
      path:
        field: layout_path
- type: symbol
  from:
    data: airports
  properties:
    enter:
      size:
        scale: size
        field: traffic.flights
      fill:
        value: steelblue
      fillOpacity:
        value: 0.8
      stroke:
        value: white
      strokeWidth:
        value: 1.5
    update:
      x:
        field: layout_x
      y:
        field: layout_y
- type: path
  name: cell
  from:
    data: airports
  properties:
    enter:
      fill:
        value: transparent
      strokeWidth:
        value: 0.35
    update:
      path:
        field: layout_path
      stroke:
        signal: cell_stroke
- type: path
  interactive: false
  from:
    data: routes
  properties:
    enter:
      path:
        field: layout_path
      stroke:
        value: black
      strokeOpacity:
        value: 0.35
- type: text
  interactive: false
  properties:
    enter:
      x:
        value: 600
      y:
        value: 0
      fill:
        value: black
      fontSize:
        value: 20
      align:
        value: right
    update:
      text:
        signal: title
```
