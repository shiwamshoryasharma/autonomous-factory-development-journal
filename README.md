# Building an autonomous factory digital twin

Early in this Accenture-sponsored project, we put a robot arm between two worktables in a simulation. We could look at the scene and discuss whether the stations belonged there. Later, with two grippers facing each other over a single part, we had a much harder question to answer. At the instant one gripper opened, what told us the other one had taken the load?

Our objective was to follow a part through printing, robot handling, CNC machining and mobile transport, and give an operator a way to understand what the factory was doing along the way. This is a developer’s account of the prototype work between November 2025 and October 2026. The small experiments described here came from testing. The main project had its own implementation.

## Getting the motion to make sense

The printer was a good place to start reasoning about motion because we could name three things and follow each one: the carriage, the nozzle and the bed. The nozzle moves along its own slide, but it also travels with the carriage. The bed moves independently. With the nozzle held still, raising the bed changes the distance to the part. That simple example gave us a precise question to ask of the model: relative to which part of the machine are we measuring movement?

The CNC made that question harder. The tool and the workpiece sit on separate branches of the machine description. One of the joints named for X travel even declares its direction along the local Y axis. Reading the label gave an incomplete account of what the table would do. We had to follow the joint back through its parent frames. A small Python inspection tool made this repeatable: it walked each machine from its root and reported broken references, duplicate links and disconnected parts. That gave us something concrete to check before interpreting motion in the scene.

## Trying the handoff from another angle

In July we kept revisiting the same small orange workpiece. From above, we could judge the gap between the part and the gripper jaws. From table height, we could see whether those jaws were actually low enough to meet it. The views came from different attempts, on different days. We could use them to choose what to inspect next, but combining them into one apparently convincing grasp would have told the wrong story.

Adding the second arm changed what we needed to observe. Even when both grippers appeared aligned, a still view left the order of attachment and release unresolved. We could describe the pose in detail and still have no evidence that the handoff completed. That gap between a plausible scene and a recorded event kept coming up elsewhere in the project.

## Debugging what the operator saw

During one browser test, the connection failed repeatedly. Cleanup then failed on a call to a termination method the client did not provide. We had evidence for a broken cleanup path. The first disconnect still needed its own explanation.

One of the more useful camera experiments was small enough to run without a camera. We published a buffering event, connected a viewer, and sent a streaming event from a worker thread. The viewer received the earlier event from recent history and the later one through the event loop. Two lines of output let us check two distinct parts of delivery. We could repeat that test independently of the browser, the network and the video decoder. Its success answered a question about the status display. Whether a frame reached the screen remained a separate test.

## Following a job through retries

One August event feed repeatedly reported a failure on the first robot while the printer waited. In a later dry run, the dashboard explicitly named clearance from the second robot as the condition holding up the next batch. These were different runs. The later message was useful because an operator could tell what the scheduler was waiting for. It could not retrospectively explain the earlier failure.

A timing question exposed a similar gap. We asked how long a product had taken, and the recorded response identified a missing start time. A finish event alone gives no duration. Once retries are involved, even two valid timestamps can produce a misleading answer if they belong to different attempts. We needed to know which attempt each timestamp described and whether its clock measured simulation time or elapsed wall time.

By October, the task form brought the product, quantity, deadline, route and two machine programs together. The printer preview let us inspect layers and surfaced a warning about priming moves. That was a concrete step forward: we could examine how a selected file was interpreted before asking what happened on the factory floor. The prototype record ends at those inspection steps. It contains no end-to-end production benchmark.

The broader work spanned NVIDIA Isaac Kit, MATLAB, National Instruments tooling, SQL, retrieval-augmented generation, agentic workflows, kernel-level development, Electron, Win32 packaging and embedded Python. The part of that experience I would carry into another project is much smaller: when a run fails, find one question that a repeatable test can answer. For us, that sometimes meant following a single joint. Other times it meant asking where one timestamp had gone.

## Prototype release

We plan to make the prototype public once the patent is granted.
