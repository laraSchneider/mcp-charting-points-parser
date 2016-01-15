# mcp-charting-points-parser
### Parses CSV files created by the Match Charting Project
Uses the Universal Match Object (UMO) https://github.com/TennisVisuals/universal-match-object to create navigable objects for each match found in MCP .csv files

#### Requirements:
- Node
- CSV point files downloaded from: https://github.com/JeffSackmann/tennis_MatchChartingProject

#### Installation
- Create project directory 'mcpParse'
- Download .zip file
- Unzip in project directory
- move MCP .csv files into mcpParse/cache/ sub-directory
*(two example files are provided, 'example' and 'testing')*

Navigate into the 'mcpParse' directory and:
```
npm install
```
#### Module Usage
Navigate to the directory above the project directory:
```
node

> p = require('./mcpParse')()

> p.parseArchive('example')
Loading File:./mcpParse/cache/example.csv
Please be patient if file is large...

Parsing CSV File...
657 points loaded
Separating Matches...
4 matches
Matches Loaded
Parsing Shot Sequences...
====
4 Matches Successfully Parsed

> p.matches.length
4
```
Each match contains tournament information as well as a UMO which can be queried/navigated using "accessors":
```
> tournament = p.matches[0].tournament
{ name: 'Tour Finals',
  division: 'M',
  date: Sun Nov 22 2015 00:00:00 GMT+0100 (CET) }

> match = p.matches[0].match
{ [Function: match]
  options: [Function],
  points: [Function],
  winProgression: [Function],
  gameProgression: [Function],
  push: [Function],
  pop: [Function],
  players: [Function],
  score: [Function],
  reset: [Function],
  sets: [Function],
  pointIndex: [Function],
  findPoint: [Function] }
```
You won't need most of these accessors for match analysis; review the REAME for the Universal Match Object if you are interested in learning more about the accessors not covered in these examples.  
```
> players = match.players()
[ 'Roger Federer', 'Novak Djokovic' ]

> match.score().match_score
'6-3, 6-4'

> match.score().winner
'Novak Djokovic'

> match.points().length
115
```

A single point looks like this:
```
> point = match.points()[0]
{ serves: [ '6' ],
  rally: [ 'b19', 'f3', 'b2', 'b1n@' ],
  terminator: '@',
  result: 'Unforced Error',
  error: 'Net',
  serve: 2,
  first_serve: { serves: [ '4n' ], error: 'Net' },
  code: '4n|6b19f3b2b1n@',
  winner: 1,
  point: '0-15',
  server: 0,
  game: 0 }
```
For **winner** and **server**,  '0' and '1' indicate the array position of the player.  The server of the point is:

```
> players[point.server]
'Roger Federer'
```

The winner of the point would be:

```
> players[point.winner]
'Novak Djokovic'
```
### Analysis & Statistics
Be sure to check out the **Analysis** and **Statistics** modules which you can read about in the documentation

### Convenience
Each match UMO can be queried to find specific points, given the set, game and score. This can be useful in debugging. Sets and Games begin at 0, so the first point of the 7th Game of the 1st Set can be accessed using the function **pointIndex(*set*, *game*)**:
```
> match.pointIndex(0,6)
42
```
If you know the point score:
```
> match.pointIndex(0,6, '30-15')
44
```
To search and display all point details, use **findPoint()**.
```
> match.findPoint(0,6,'15-0')
{ serves: [ '4' ],
  rally: [ 'f29', 'f3*' ],
  terminator: '*',
  result: 'Winner',
  serve: 1,
  code: '4f29f3*|',
  winner: 0,
  score: '15-0',
  set: 0,
  server: 0,
  game: 6 }
```
You can use an optional final parameter to make the search 'lazy', which will search for '15-0' as well as '0-15'.
```
> match.findPoint(0,6,'15-0', true)
...
```

### Point Translation
Several convenience functions are provided for working with MCP match data.

**decipherPoint()** provides an english-language translation of a point.
```
> p.decipherPoint(point)
[ 'T Serve',
  'Backhand cross-court; Close to Baseline',
  'Forehand down the line',
  'Backhand to the middle',
  'Backhand to the left side; Netted; Unforced Error' ]
```
**decipherSequence()** provides an english-language translation of an MCP shot sequence. An optional second argument enables passing the point score (e.g. '0-15') which aids in determining the trajectory of the return of service.
```
> p.decipherSequence('6f=37b+3b3z#', '0-15')
[ 'T Serve',
  'Forehand at the Baseline to the right side; Within Service Boxes',
  'Backhand approach shot cross-court',
  'Backhand cross-court',
  'Backhand Volley; Forced Error' ]
```
**decipherShot()** provides an english-language translation of a single MCP shot. The point score is an optional parameter.
```
> p.decipherShot('6*', '0-15')
{ sequence: 'T Serve; Ace',
  full_sequence: 'T Serve, Winner',
  direction: 1 }
  ```
