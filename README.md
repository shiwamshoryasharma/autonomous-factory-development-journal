# Building an autonomous factory digital twin

*Developer journal · November 2025 – October 2026*

On July 15, I could look down at an orange workpiece and see daylight on either side of it, between the part and the gripper jaws. Three days later, a view from table height made a different question easy to ask. Were the jaws low enough to pick it up? I keep coming back to those two attempts because they explain much of what it was like to develop this prototype. Both views helped. Neither one recorded a pickup.

We set out to follow a part through printing, robot handling, CNC machining and mobile transport in an Accenture-sponsored autonomous factory project. An operator should be able to ask where the part is, which machine has it and what it is waiting for. Those sound like ordinary questions until two machines are moving, a viewer has just reconnected and the same job has already been tried once.

This journal brings together two records: what we could see during individual tests, and what I remember building with the machine teams. The code in the résumé prototype is a small testing layer. The wider project had a different implementation, and some of the most important decisions came after we had already invested time in an approach.

## The development timeline

The dated entries below come from our testing notebook. The four phases follow the engineering decisions; several teams worked on their machines at the same time.

> **◆　PHASE 01　·　Teaching the robot**<br>
> **[01　The training cells](#one-arm-and-two-tables)** — One arm and two tables became a repeated Isaac Lab environment for object-detection-based pick-and-place.<br>
> **[02　The change of direction](#reaching-through-the-opening)** — New project criteria retired the training approach and removed the camera mounted on the arm.
>
> **◆　PHASE 02　·　Making the part behave**<br>
> **[03　Six printer concepts](#six-concepts-for-one-printed-part)** — Fleeting droplets became persistent layers, then a performance and physics problem we had to solve together.<br>
> **[04　Two arms and a CNC](#the-orange-workpiece)** — Pickup, handover and placement had to work before the CNC could change the printed geometry.
>
> **◆　PHASE 03　·　Seeing what happened**<br>
> **[05　Streaming and heat](#when-the-viewer-disconnected)** — The Kit stream, thermal simulation and operator view gave us different pieces of evidence.<br>
> **[06　Failures and waiting](#the-clock-and-the-waiting-robot)** — A missing start time and a clearance wait showed where a machine’s local status ran out of context.
>
> **◆　PHASE 04　·　Running one factory**<br>
> **[07　The wrong machine](#when-a-print-request-reached-the-robot)** — An agent could receive a print request and choose a pickup as its first action.<br>
> **[08　Bringing the teams together](#the-machine-teams-and-the-factory-brain)** — Machine backends, the factory-wide algorithms, AWS hosting and the combined control extension had to work as one system.

## One arm and two tables

On November 3 there was one robot arm between two worktables. Four days later, several copies of the cell occupied the same scene. Those tables and repeated robots belonged to our first Isaac Lab experiment. We created a model to teach the robot pick-and-place using object detection. The repeated cells make much more sense when you know what we were trying to teach.

At that point the teaching problem and the factory problem still seemed close. A camera mounted on the robot arm was part of how we expected the robot to find its target. If you had seen only those early scenes, you could reasonably have assumed the work that followed was a matter of refining that model and adding more stations. The project took a different turn.

## Reaching through the opening

The project criteria changed and we introduced a new architecture. We dropped the earlier teaching approach. The revised robot design removed the arm-mounted camera and the object-detection step that depended on it. The underlying requirement survived the change: the robot still needed to pick up an object whose position could vary. That was the expensive part of the pivot. We had lost an assumption that the first model was built around, and still owed the factory the same capability.

By January, the arm was reaching toward a machine enclosure. The opening gave the wrist and tool a much smaller region to pass through. Looking only at the tip was misleading. It could appear to reach the desired point while the wrist still occupied space we needed to inspect near the machine wall.

In March, we had an RViz planning view with the robot, its current state, a goal and the planning settings together. Another attempt placed the arm beside a round table and exposed part of the graph driving the scene. I could ask a more precise question of each attempt. Which pose was the planner being asked to reach? Where did the tool appear relative to the tabletop? A snapshot of the planning panel could answer the first question. Establishing clearance throughout the approach would require the trajectory between the poses as well.

That distinction affected how I read the rest of our notes. A useful test record needs enough context for someone else to tell what we asked the system to do. A convincing final pose can otherwise hide almost everything that happened on the way there.

## Which frame is moving?

In April, the floor plan put the printer, CNC, two robots and mobile-robot stops on one page. In May, an operator panel exposed individual joint positions. The floor plan made the handoff route easy to discuss. The joint panel let us look at what one machine was actually doing. The machine descriptions in the code connected those two scales.

The printer is the example I would use to explain the problem to another developer. Its nozzle sits on a slide carried by the carriage. Move the carriage and the nozzle goes with it, even if the nozzle’s own joint value stays constant. The bed moves on a separate branch. Raising it changes the nozzle-to-part distance while the nozzle can stay at exactly the same position in the room. Before drawing a deposited line, I need to know which of those positions my trace represents.

The CNC made reading the model by eye even harder. Its spindle and table belong to separate branches, and a joint with X in its name declares its direction along a local Y axis. The mobile robot offered another trap: both wheel joints use the same local axis, but one wheel’s frame is turned around relative to the other. The little Python inspection tool in the prototype gave us a repeatable starting point. It followed links from the root and checked for duplicate names, broken references and unreachable parts. Once that check passed, we could turn to the separate work of examining axes, physical dimensions and clearance.

## Six concepts for one printed part

On the printer, our first attempt at layer-by-layer deposition looked like a trail of droplets. A point would appear as the nozzle advanced, remain for a short time and disappear. By the time the next stretch was being drawn, an earlier stretch was already gone. We could watch where the nozzle had just been. We could barely inspect what it had built.

On the second attempt the 3D printer team and I changed how we represented deposited material on the bed. This time a layer stayed where we had placed it. That was progress I could see immediately. The price showed up as the print continued: frames dropped and the physics simulation struggled to keep running smoothly. A short test of whether one layer persisted could pass while a longer print was still uncomfortable to run. From then on, both observations mattered to us.

It took six concepts and a lot of retries. By the sixth, the printer team and I had resolved the disappearing layers, the frame drops and the disruption to the physics flow that we had seen in the earlier versions. That was joint work. I remember the second attempt particularly well because its apparent success gave us the next, harder problem. We had finally made enough material stay on the bed to discover what keeping it there cost.

## The orange workpiece

At the end of June, a side view put the wrist close to an open machine. In early July, we also had two physical-camera views of an arm, taken eleven seconds apart. One crop showed very little around the wrist. The other included more of the arm and cabling. Changing the viewpoint bought us context before we changed anything in the model.

The July 15 and July 18 attempts with the orange part are the clearest example. The overhead view let me inspect lateral alignment. The side view gave me the table as a height reference. The dates mattered as much as the camera positions. These were separate attempts. I could use the second to decide what to check on a future run. I could not use it as the missing second angle of the first run.

The failures looked different from one attempt to the next. Sometimes the simulated robot hit what looked like an invisible wall. Sometimes it moved in a direction I hadn't expected. On other attempts the jaws reached the part and closed, but friction and contact still failed to hold it. Reaching the pickup pose, closing the gripper and carrying the part away were three separate things I had to watch. Getting the first two right made the third failure particularly frustrating.

We tried several approaches after losing the arm-mounted detection system. I eventually got adaptive pickup and placement working without feeding the robot a set of joint positions for every new target location. I used NVIDIA cuMotion in Isaac Sim and MoveIt 2 inverse kinematics on the physical robot. For this particular job, getting the simulation to behave was the harder part. The physical arm was comparatively straightforward. That experience was almost the reverse of what we would encounter later when we connected the whole factory.

On July 23 the orange part sat between the two opposing grippers. The image preserves one pose. My responsibility extended across what happened on either side of that pose: I designed the adaptive pickup, the handover between the two arms and the placement inside the CNC. The exact way the robot adapts to a new position, transfers the part and completes the placement remains private.

## Cutting into a part we had just printed

The CNC team and I inherited an awkward geometry problem from the printer. Our stock was a block built up from deposited layers. If we wanted a hole through that block, which faces had to disappear? A hole visible on the top surface could still leave material spanning the opening on a layer underneath. We needed the changed geometry to make sense through the depth of the cut.

We worked through how to remove faces within the printed layers and represent the new configuration. Together, the CNC team and I got that working. It changed how I thought about the boundary between the two machines. For the printer, a completed layer was material we had worked hard to preserve. For the CNC, some of that same material now had to be removed deliberately.

The autonomous mobile robot (AMR) was the most straightforward machine in my part of the journey. The AMR team owned its machine work. Where it mattered to me was in the combined flow: a transport step still had to fit the sequence the other teams were building.

## When the viewer disconnected

I owned the NVIDIA Kit streaming work. Once we started streaming the scene and building the thermal flow, a report that “the printer looks wrong” could refer to several places. Was the printer behaving incorrectly in the simulation? Was the thermal view showing the wrong thing? Had the browser simply stopped receiving frames? Those questions belonged to different parts of the stack, but to an operator there was only one screen.

On August 13, the browser repeatedly failed to connect its WebSocket. Cleanup also produced a concrete error: the client tried to call a termination method that was unavailable on the object it held. That second failure was specific enough to investigate in the client. It still left the original disconnect unexplained. Treating the last error on screen as the cause of the first one would have sent the investigation in a circle.

The camera-event excerpt captures a smaller problem that we could reason through completely. First, it publishes a buffering status. A viewer subscribes afterwards and receives that earlier status from recent history. Then a worker thread schedules a streaming update on the event loop, and the same viewer receives it live. I like this example because the two lines of output look nearly identical, yet they reach the viewer by different paths. Seeing both in order checks something specific about subscription and delivery.

There is a deliberate trade-off in that code. Each viewer has a bounded queue. When a slow subscriber falls behind, the oldest pending status makes room for the latest one. That suits an operator reopening a status panel who needs to know what is happening now. It would be a poor source for reconstructing every event in a failed run. And even a perfectly delivered *streaming* status only tells us what the worker reported. To establish that the operator saw video, we would need an observation at the browser end of the path.

In August we also had a view inside the simulated printer chamber and, a day later, a thermal rendering with separate nozzle, bed and chamber readings. Building the thermal flow was another difficult part of the project. A colored bed told us what the renderer displayed at one instant. Following the readings as the process advanced was a different question, and comparing them with physical measurements would require a separate record.

## The clock and the waiting robot

On August 25, we asked a plain question: how long did the product take in the simulation, and how did that compare with real time? The recorded response identified the missing start time. I would rather have that gap visible than publish a number nobody could reproduce. A finish timestamp tells us when something ended. To calculate the duration, we still need the start of the same attempt and a clock whose meaning we know.

Retries add an especially awkward failure mode. Pair the start of the first attempt with the finish of the second, and the result can look perfectly reasonable. The arithmetic can be correct while the number answers the wrong question. That is why an attempt identifier belongs alongside the timestamps. Comparing wall time with simulation time also requires keeping both clocks explicit.

On August 31, the event feed repeatedly reported an R1 failure while the printer waited. The repetition showed the symptom was recurring, but it did little to explain the dependency. A September 1 dry-run dashboard offered a more useful example of reporting: the batch was waiting for clearance from the second robot, R2, and the interface exposed the cooldown. Someone reading that state could identify what had to change before the batch could advance. It was a different run, so it would be a mistake to use it to diagnose the previous day’s R1 failure.

## From a request to a toolpath

By October, the task form let us bring the product, quantity, deadline and route together with separate program selections for the printer and CNC. Keeping those two selections visible mattered. If a preview looked unexpected, we could first ask which file had been selected for which machine. That is a much cheaper question to resolve than debugging machine motion from the wrong program.

The October 25 printer preview exposed the layer count, G-code line count and a warning about priming moves. That warning is one of the details I value most in the late prototype. If a movement was omitted from the preview, the interface gave us a place to start looking for the reason. To compare the preview with the simulated printer, we would still need the same file revision and layer. To say what happened to a physical part, we would also need the assignment and completion record for that run. That October record is useful to me because it preserves exactly what the operator could inspect before asking the factory to act.

## When a print request reached the robot

People would ask the agent to print an STL, STEP or G-code file. In some tests it ignored the file. In others it interpreted a printing request as an instruction for robot R1 to pick up a part. Imagine standing at the cell after asking for a print and being offered a pickup as the first action. The robot might be available and pickup might belong later in the route. Neither fact makes it the right machine or action at that moment.

Those failures explain why connecting an LLM to our factory took more work than putting a chat panel over it. An STL or STEP file describes geometry. A G-code file already contains machine instructions. The operator's wording could sound the same for all three, but the state of the work was different. I had to think about what the operator had actually supplied, what stage the job was in, and which machine could meaningfully act next. The task form and program previews gave us concrete selections against which to compare an agent's interpretation.

We also had a custom machine-learning model for predicting machine health. Getting its predictions into the AI workflow took repeated integration attempts. Producing a prediction inside one component and making that result available in the context of the right machine were separate milestones. It was one more case where a component could do its own job and the operator could still be missing the answer they needed.

## The machine teams and the factory brain

The hardest architectural work for me was giving each machine its own “brain” and then getting those brains to participate in one flow. I thought of them as children coordinated by a parent. A machine knows about its own task and condition. The parent needs enough context across machines to know why a robot is waiting for a printer, or why a pickup must wait until the next station is clear. I created the combined architecture, the agentic AI workflow and the main factory brain that connects those machine-level decisions to execution.

I also wrote the factory algorithms: the genetic algorithm (GA), particle swarm optimization (PSO) and ant colony optimization (ACO). They were part of my work on the factory-wide system alongside the AI integration. The flow posed a different engineering problem from getting any one machine to run. A locally sensible action, like telling an idle robot to pick, could still be wrong for the job as a whole.

The machine backends were shared work. Roughly half of each individual backend came from its respective machine team. I owned the robot side and its custom extension. The CNC and other machine teams developed their own extensions, and I wrote the combined control extension that brought them together. I also handled the AWS API, hosting and Kit streaming. The wider project included Win32 packaging, Electron, embedded Python, kernel-level work, MATLAB, National Instruments tooling, SQL and retrieval-augmented generation. Those were parts of a larger team effort.

The distinction between simulation and hardware depended on the problem in front of us. **For robot pickup, simulation was harder. For the integrated factory, running the real machines together was the hardest part.** Once each station had an owner and its own working behavior, we still had to finish the sequence across all of them. We got there with the 3D printer team, my robot work, the CNC team and the AMR team. I take particular pride in designing the architecture that let us bring those contributions together.

## Prototype release

We plan to make the prototype public once the patent is granted.
