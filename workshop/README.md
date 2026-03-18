# Tracks in the Dark - Workshop

Welcome to the workshop repository for the Tracks in the Dark hackathon regarding Graphs.


## Basic Graph Implementation

Let us begin with implementing a basic graph in TypeScript.
The first step is the ability to identify a single node in the graph.
A `type` is created to give a name to the identifier, which is an mapping for a `string` type.
This way, it is not required to remember which type was assigned as an Id,
since the language server will provide this information for us.
It also reads better, but that's personal preference.

```ts
export type NodeId = string;
```

The next step is to create the graph object.
As said before, a graph contains nodes and edges, so let's model them.
Since the structure of the data is being defined, an `interface` is used.
A class is overkill, we are only defining structure, not behaviour.
A type is designed for complexer structures or type mappings.

```ts
// A node is a single point on the graph
export interface Node {
    id: NodeId;
}

// An edge is a connection between two nodes
export interface Edge {
    from: NodeId;
    to: NodeId;
}
```

The basic structure is ready, now the components need to come together.
A class is made to represent a graph. Maps are used to store the components.
First, a nodes field is introduced storing the individual nodes,
secondly, a connections list is introduced. This concept is called an `adjacency list`.
The nice thing about maps is the ability to use the `set` and  `get` methods.
The positions are calculated rather than looked-up, which makes the structure whicked fast.
If you are interested in why this is, look for [Hash Maps]https://en.wikipedia.org/wiki/Hash_table).
In addition, get and set just reads better than endless loops 😂.

It does require some explanation on the `?` and `??` operators.
Both operators are used when it is possible to get an `undefined` value.
The `addEdge` function below uses the `?` operator to determine whether the node exists in the list.
If it exists, the push is executed. If it does not exist, nothing happens.
The `??` operator is covered when it appears in code. 

```ts
export class Graph {
    // The components of the graph
    private nodes = new Map<NodeId, Node>();
    private connections = new Map<NodeId, Edge[]>();

    // Adding a node to the graph
    addNode(id: NodeId): void {
        this.nodes.set(id, { id });
        this.connections.set(id, []);
    }

    // Adding a connection between to nodes
    addEdge(from: NodeId, to: NodeId): void {
        this.connections.get(from)?.push({ from, to });
    }
}
```

Finally, it is important that we make sure this works.
Therefore, a method will be added that can be called later.
This method returns the list of all edges for a specific node.
It uses the `??` operator, if the operation on the left of the operator returns undefined, 
the value on the right is returned instead. In other words:
if the provided id is not present in the list of connections, an empty list is returned instead.

```ts
// Getting the edges for a specific node
getConnections(id: NodeId): Edge[] {
    return this.connections.get(id) ?? [];
}
```


## Visualization of the Graph

Having a graph is nice, though it is important that one can actually visualize it.
A graph is a collection of nodes connected by edges.
This means nodes are drawn, followed by the the edges in between.
Just... where should the nodes be drawn?
Without this information, all nodes are going to be drawn on the top-left of the screen.
For this, a position is used, an x-coordinate and a y-coordinate.
This interface would look like this:

```ts
export interface Position {
    x: number;
    y: number;
}

export interface Node {
    id: NodeId;

    // Add the position to the Node interface
    position: Position;
}
```

Now that the interface is ready, the nodes can be drawn onto the screen.
There are several ways to tackle this problem.
To keep things simple, the `graph` class is used.
The assumption is made that the graph is going to be drawn onto a `<canvas>`-element.
For starters, let's make the canvas black after retrieving the `context`.
The `fillStyle` property can be used to change the color of shapes.

```ts
render(canvas: HTMLCanvasElement): void {
    const context = canvas.getContext('2d')!;

    // Draw the background
    context.fillStyle = 'black';
    context.fillRect(0, 0, canvas.width, canvas.height);
}
```

After setting up the base function, and getting the canvas a nice dark color,
it is time to draw the actual nodes, they will be colored orange.
All the nodes are retrieved using the `values()` method.
Then a circkel is drawn at the position of the node.

```ts
render(canvas: HTMLCanvasElement): void {
        const context = canvas.getContext('2d')!;

        // Drawing the background
        context.fillStyle = 'black';
        context.fillRect(0, 0, canvas.width, canvas.height);

        // Draw the nodes
        context.fillStyle = 'orange';
        for (const node of this.nodes.values()) {
            context.beginPath();
            context.arc(node.position.x, node.position.y, 10, 0, Math.PI * 2);
            context.fill();
        }
    }
```

Now there are circkels on the screen, neat.
Time to get the edges in between.
The edges are drawn before the nodes, to make sure the nodes are drawn on top of the lines instead of the other way around.
The style and width are set, then alle the edges are gathered, per node.
Each edge is drawn by moving the cursor inside the canvas to the `fromNode` and drawing a line to the `toNode`.
This adds the different edges.

```ts
render(canvas: HTMLCanvasElement): void {
        const context = canvas.getContext('2d')!;

        // Drawing the background
        context.fillStyle = 'black';
        context.fillRect(0, 0, canvas.width, canvas.height);

        // Draw the edges
        context.strokeStyle = 'white';
        context.lineWidth = 2;
        for (const edges of this.connections.values()) {
            for (const edge of edges) {
                const fromNode = this.nodes.get(edge.from)!;
                const toNode = this.nodes.get(edge.to)!;

                context.beginPath();
                context.moveTo(fromNode.position.x, fromNode.position.y);
                context.lineTo(toNode.position.x, toNode.position.y);
                context.stroke();
            }
        }

        // Draw the nodes
        context.fillStyle = 'orange';
        for (const node of this.nodes.values()) {
            context.beginPath();
            context.arc(node.position.x, node.position.y, 10, 0, Math.PI * 2);
            context.fill();
        }
    }
```

**Optionally**, the graph made is a directed graph, with a start and end node.
To indicate the direction of an edge, an arrow can be added to the center of the line.
For this, the center point of the line needs to be calculated.
The difference in position can be used to get the angle between the two nodes.
To cheat, and prevent a lot of extra calculation, the content of the canvas can be moved instead.
First the current position of the content is saved throught the `save()` method.
Then, the center of an edge is placed at the `(0, 0)` position, at a straight angle.
Now drawing the actual traingle is simple.
Afterwards, the position of the content needs to be restored through the `restore()` method.

```ts
render(canvas: HTMLCanvasElement): void {
    const context = canvas.getContext('2d')!;

    // Drawing the background
    context.fillStyle = 'black';
    context.fillRect(0, 0, canvas.width, canvas.height);

    // Draw the edges
    context.strokeStyle = 'white';
    context.fillStyle = 'white';
    context.lineWidth = 2;
    for (const edges of this.connections.values()) {
        for (const edge of edges) {
            // Draw line between nodes
            const fromNode = this.nodes.get(edge.from)!;
            const toNode = this.nodes.get(edge.to)!;

            context.beginPath();
            context.moveTo(fromNode.position.x, fromNode.position.y);
            context.lineTo(toNode.position.x, toNode.position.y);
            context.stroke();

            // Draw arrowhead in the middle of the edge
            const middleX = (fromNode.position.x + toNode.position.x) / 2;
            const middleY = (fromNode.position.y + toNode.position.y) / 2;
            const differenceX = toNode.position.x - fromNode.position.x;
            const differenceY = toNode.position.y - fromNode.position.y;
            const angle = Math.atan2(differenceY, differenceX);

            context.save();
            context.translate(middleX, middleY);
            context.rotate(angle);
            context.beginPath();
            context.moveTo(5, 0);
            context.lineTo(-5, -5);
            context.lineTo(-5, 5);
            context.closePath();
            context.fill();
            context.stroke();
            context.restore();
        }
    }

    // Draw the nodes
    context.fillStyle = 'orange';
    for (const node of this.nodes.values()) {
        context.beginPath();
        context.arc(node.position.x, node.position.y, 10, 0, Math.PI * 2);
        context.fill();
    }
}
```