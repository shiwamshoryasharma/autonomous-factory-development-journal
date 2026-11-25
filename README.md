# Building an autonomous factory digital twin

When I joined the Accenture-sponsored autonomous factory project, the work was already underway. One of the first problems I worked on was teaching a robot to pick up a part from a table. By the time we were connecting the whole factory, I was debugging almost the opposite problem. An operator would ask the agent to print a part, and sometimes the first thing it wanted to do was send a robot to pick it up. There was nothing to pick up yet.

Between those two problems lay most of my work on the project. I designed the robot handling layer, worked with the printer and CNC teams on what happened to the part inside their machines, and brought the individual machine backends together. I wanted an operator to be able to give the factory a job and follow the same part through printing, handling, machining and transport. Every team owned a piece of that journey. I owned how those pieces worked together.

> **01 · [Teaching the robot](#teaching-the-robot)**<br>
> The arm-mounted camera was central to our first approach. Then the project changed direction.
>
> **02 · [Making the part](#making-the-part)**<br>
> Getting a printed layer to stay on the bed solved one problem and exposed the next.
>
> **03 · [Following a run](#following-a-run)**<br>
> Streaming the simulation and reporting machine state gave us two different ways to lose track of a job.
>
> **04 · [Running one factory](#running-one-factory)**<br>
> The agent had to understand what an operator supplied, which machine could use it and when.

## Teaching the robot

We started in Isaac Lab with one robot, two worktables and a model for teaching pick-and-place through object detection. We repeated the training cell so we could work on the same task across several instances. The camera mounted on the arm was part of the premise. We expected the robot to use it to locate the object it needed to pick up.

Then the project criteria changed. We introduced a new architecture and dropped the arm-mounted detection approach. The robot still had to handle a part whose position could vary. Giving it a fresh list of joint positions every time the part moved would have pushed the work back onto the operator. Finding another way to meet that requirement took several attempts.

Some failures were easy to describe and difficult to explain. The simulated arm would stop as if it had struck an invisible wall. On another attempt it would move in a direction I hadn't expected. Sometimes it reached the part and closed its jaws exactly where I wanted, only to lose its hold as it lifted. That last failure made me pay closer attention to friction and contact. Watching where the robot ended up told me very little about the moment the part slipped.

I eventually got adaptive pickup and placement working with NVIDIA cuMotion in Isaac Sim and MoveIt 2 inverse kinematics on the physical robot. For pickup, the physical arm was the more straightforward of the two. Getting the simulated gripper to approach, make contact and hold the part reliably took more of my time.

The next move involved two arms. One had to hand the part over while the other took its weight, and then the part had to go inside the CNC enclosure. At the opening, the tool could reach its target while the wrist still needed clearance from the wall. I designed the pickup, the handover and the placement inside the CNC. I'm keeping the details of how the robot adapts to different part positions private until the prototype is released.

## Making the part

The printer gave us a particularly frustrating first result. The nozzle moved along its path, and little droplet-like points appeared behind it. Each point lasted a moment and disappeared. By the time the nozzle reached the next stretch, the material it had just deposited was already gone. We had reproduced the motion of printing, but we still couldn't build up a part.

For the second concept, the 3D printer team and I changed how we kept deposited material on the bed. The earlier layers finally stayed in place as the nozzle moved on. A longer print exposed the cost. Frames started dropping, and the physics simulation struggled to advance smoothly. It was tempting to call that version a success after the first layer. Letting the job run was what showed us how much work remained.

We went through six concepts. By the sixth, the printer team and I had resolved the disappearing layers, frame drops and disruption to the physics simulation that we had struggled with in the earlier versions. Of all those attempts, the second one sticks with me. Keeping the material on the bed was exactly what we wanted, and doing it successfully was what made the performance problem visible.

Once the printed block reached the CNC, the CNC team and I had the reverse problem. We had put real effort into preserving each deposited layer. Now we needed to remove faces from those layers to create a different geometry. Take a hole through a printed block. Opening the top surface leaves the interior of the block to deal with. If material still spans the opening farther down, the cut is incomplete. We worked through how to represent that change through the depth of the part and got it working together.

The autonomous mobile robot was the most straightforward machine from my side of the project. The AMR team handled its machine-specific work. My part was fitting transport into the sequence so it happened when the part was ready to move.

## Following a run

I was responsible for streaming NVIDIA Kit and for the AWS API and hosting. One browser problem gave me two failures to untangle. The WebSocket kept failing to connect, and while cleaning up the failed session, the client tried to call a termination method that didn't exist on the object it held. The cleanup error was specific enough to investigate immediately. It still left me with the original question of why the connection had dropped.

One small camera test helped isolate what our status messages actually proved. We published a buffering event, connected a viewer, and sent a streaming update from a worker thread. The viewer received the older event from recent history and the newer one from the running event loop. That gave us a repeatable check for a reconnecting viewer. Whether the browser had actually received and displayed a frame was another part of the path to check.

Building the thermal flow added to the challenge. The nozzle, bed and chamber each had their own simulated reading. If an operator reported that the printer appeared to have stopped heating, both the simulation and the stream deserved attention. A frozen browser could leave a changing temperature looking perfectly steady. Getting the values through to the operator mattered as much as drawing the thermal view.

We ran into a similar problem when we asked how long a product had taken. We had a finish time and were missing the start. Retrying a job makes that surprisingly easy to get wrong even when both timestamps exist. Pair the beginning of one attempt with the end of another and the arithmetic works. The number is still useless to the person asking how long their run took. We needed to know which attempt each event belonged to and whether we were reading simulation time or elapsed time on the factory floor.

## Running one factory

“Print this file” sounds like a simple request. An operator might supply an STL, a STEP file or G-code. The first two describe geometry, while G-code already contains machine instructions. In our tests the agent sometimes ignored the file. Other times it assigned a pickup to R1. Picking up the part belonged later in the route, so the response could sound sensible until you asked what the robot was supposed to pick up.

I built the agentic workflow around that larger problem of execution. Each machine needed its own controller, or “brain,” with an understanding of its local work. I designed the main factory brain to coordinate those controllers. The relationship I had in mind was a parent working with its children. The printer could report its state, the robot could report that it was available, and the factory still had to decide whether this particular part was ready for a handover.

The same issue came up when we connected our custom machine-health prediction model to the AI. Getting the model to produce a prediction was one step. Making that prediction useful for the machine involved in the current job took repeated integration attempts. We also had to make waiting understandable. A printer waiting for the second robot to clear a station and a printer waiting because the first robot had failed might both look idle to an operator. Those situations call for different responses.

This is where the contributions of the teams came together. The respective machine teams wrote roughly half of each individual backend. I owned the robot work and its custom extension. The CNC and other machine teams developed their machine-specific extensions, and I wrote the combined control extension. I also designed the shared architecture and wrote the main factory brain and the GA, PSO and ACO algorithms: genetic algorithm, particle swarm optimization and ant colony optimization. The wider project brought in embedded Python, Win32 packaging, Electron, kernel-level work, MATLAB, National Instruments tooling, SQL and retrieval-augmented generation.

On the robot, I had spent more time getting pickup to behave in simulation than on the physical arm. Across the factory, the balance flipped. Running the real printer, robots, CNC and AMR as one system was the hardest part. We finished that integration with the help of all the machine teams. What I'm proudest of is the work between their machines. The printer team could make a part, the robot could take it, the CNC team could change it and the AMR team could move it. My job was to make those steps belong to the same job from beginning to end.

## My Work flow documention release

I want to make the journal documentation public after the patent is awarded. There, you can see some images and videos outcomes, but not the whole output. 
