# Building an autonomous factory digital twin

*Developer journal · November 2025 – October 2026*

On July 15, I could look down at an orange workpiece and see daylight on either side of it, between the part and the gripper jaws. Three days later, a view from table height made a different question easy to ask. Were the jaws low enough to pick it up? I keep coming back to those two attempts because they explain much of what it was like to develop this prototype. Both views helped. Neither one recorded a pickup.

We set out to follow a part through printing, robot handling, CNC machining and mobile transport in an Accenture-sponsored autonomous factory project. An operator should be able to ask where the part is, which machine has it and what it is waiting for. Those sound like ordinary questions until two machines are moving, a viewer has just reconnected and the same job has already been tried once.

This journal follows the testing code and the development notes we kept along the way. The main project used its own implementation. The story here ends with what we could inspect and reproduce in the prototypes.

## The development timeline

> **● NOV 2025**　[One arm and two tables](#one-arm-and-two-tables)<br>
> We built the smallest scene in which station placement mattered.
>
> **● JAN – MAR 2026**　[Reaching through the opening](#reaching-through-the-opening)<br>
> An enclosure turned a layout question into a motion-planning question.
>
> **● APR – MAY**　[Which frame is moving?](#which-frame-is-moving)<br>
> The floor plan, the joint panel and the machine descriptions gave us three ways to read a pose.
>
> **● JUN – JUL**　[The orange workpiece](#the-orange-workpiece)<br>
> We changed viewpoints and tried to pin down the instant a part changed hands.
>
> **● AUG**　[When the viewer disconnected](#when-the-viewer-disconnected)<br>
> A browser failure led us down two different debugging paths.
>
> **● AUG – SEP**　[The clock and the waiting robot](#the-clock-and-the-waiting-robot)<br>
> A missing start time and a stalled batch exposed gaps in task reporting.
>
> **● OCT**　[From a request to a toolpath](#from-a-request-to-a-toolpath)<br>
> We could finally inspect a selected program alongside the task that requested it.

## One arm and two tables

The November 3 scene was spare. One arm stood between two work surfaces. It gave us somewhere to put a part and somewhere else the part might go. Four days later, we had several copies of the arrangement in the same scene. The repetition made spacing easier to judge. It also made it surprisingly easy to look at the layout and imagine a production sequence that we had yet to demonstrate.

I find it useful to remember how little that early scene had to answer. Nobody needed a cycle-time estimate to discuss whether the stations were sensibly arranged. A clear view of the tables was enough. The harder questions arrived as we added things with an inside and an outside, moving joints, and equipment that had to wait its turn.

## Reaching through the opening

By January, the arm was reaching toward a machine enclosure. The opening gave the wrist and tool a much smaller region to pass through. Looking only at the tip was misleading. It could appear to reach the desired point while the wrist still occupied space we needed to inspect near the machine wall.

In March, we had an RViz planning view with the robot, its current state, a goal and the planning settings together. Another attempt placed the arm beside a round table and exposed part of the graph driving the scene. I could ask a more precise question of each attempt. Which pose was the planner being asked to reach? Where did the tool appear relative to the tabletop? A snapshot of the planning panel could answer the first question. Establishing clearance throughout the approach would require the trajectory between the poses as well.

That distinction affected how I read the rest of our notes. A useful test record needs enough context for someone else to tell what we asked the system to do. A convincing final pose can otherwise hide almost everything that happened on the way there.

## Which frame is moving?

In April, the floor plan put the printer, CNC, two robots and mobile-robot stops on one page. In May, an operator panel exposed individual joint positions. The floor plan made the handoff route easy to discuss. The joint panel let us look at what one machine was actually doing. The machine descriptions in the code connected those two scales.

The printer is the example I would use to explain the problem to another developer. Its nozzle sits on a slide carried by the carriage. Move the carriage and the nozzle goes with it, even if the nozzle’s own joint value stays constant. The bed moves on a separate branch. Raising it changes the nozzle-to-part distance while the nozzle can stay at exactly the same position in the room. Before drawing a deposited line, I need to know which of those positions my trace represents.

The CNC made reading the model by eye even harder. Its spindle and table belong to separate branches, and a joint with X in its name declares its direction along a local Y axis. The mobile robot offered another trap: both wheel joints use the same local axis, but one wheel’s frame is turned around relative to the other. The little Python inspection tool in the prototype gave us a repeatable starting point. It followed links from the root and checked for duplicate names, broken references and unreachable parts. Once that check passed, we could turn to the separate work of examining axes, physical dimensions and clearance.

## The orange workpiece

At the end of June, a side view put the wrist close to an open machine. In early July, we also had two physical-camera views of an arm, taken eleven seconds apart. One crop showed very little around the wrist. The other included more of the arm and cabling. Changing the viewpoint bought us context before we changed anything in the model.

The July 15 and July 18 attempts with the orange part are the clearest example. The overhead view let me inspect lateral alignment. The side view gave me the table as a height reference. The dates mattered as much as the camera positions. These were separate attempts. I could use the second to decide what to check on a future run. I could not use it as the missing second angle of the first run.

On July 23, the part sat between opposing grippers. With one arm, I had been asking whether the jaws were in a plausible place. With two arms, I wanted to know which one carried the part when the first gripper released. Several sequences could pass through the same visible pose. To distinguish them, I would want the attachment and release events from one attempt on the same time base. That is where the handoff record leaves us. We had a much better question than we started with, and the still view could take us no further.

## When the viewer disconnected

On August 13, the browser repeatedly failed to connect its WebSocket. Cleanup also produced a concrete error: the client tried to call a termination method that was unavailable on the object it held. That second failure was specific enough to investigate in the client. It still left the original disconnect unexplained. Treating the last error on screen as the cause of the first one would have sent the investigation in a circle.

The camera-event excerpt captures a smaller problem that we could reason through completely. First, it publishes a buffering status. A viewer subscribes afterwards and receives that earlier status from recent history. Then a worker thread schedules a streaming update on the event loop, and the same viewer receives it live. I like this example because the two lines of output look nearly identical, yet they reach the viewer by different paths. Seeing both in order checks something specific about subscription and delivery.

There is a deliberate trade-off in that code. Each viewer has a bounded queue. When a slow subscriber falls behind, the oldest pending status makes room for the latest one. That suits an operator reopening a status panel who needs to know what is happening now. It would be a poor source for reconstructing every event in a failed run. And even a perfectly delivered *streaming* status only tells us what the worker reported. To establish that the operator saw video, we would need an observation at the browser end of the path.

## The clock and the waiting robot

On August 25, we asked a plain question: how long did the product take in the simulation, and how did that compare with real time? The recorded response identified the missing start time. I would rather have that gap visible than publish a number nobody could reproduce. A finish timestamp tells us when something ended. To calculate the duration, we still need the start of the same attempt and a clock whose meaning we know.

Retries add an especially awkward failure mode. Pair the start of the first attempt with the finish of the second, and the result can look perfectly reasonable. The arithmetic can be correct while the number answers the wrong question. That is why an attempt identifier belongs alongside the timestamps. Comparing wall time with simulation time also requires keeping both clocks explicit.

On August 31, the event feed repeatedly reported an R1 failure while the printer waited. The repetition showed the symptom was recurring, but it did little to explain the dependency. A September 1 dry-run dashboard offered a more useful example of reporting: the batch was waiting for clearance from the second robot, R2, and the interface exposed the cooldown. Someone reading that state could identify what had to change before the batch could advance. It was a different run, so it would be a mistake to use it to diagnose the previous day’s R1 failure.

## From a request to a toolpath

By October, the task form let us bring the product, quantity, deadline and route together with separate program selections for the printer and CNC. Keeping those two selections visible mattered. If a preview looked unexpected, we could first ask which file had been selected for which machine. That is a much cheaper question to resolve than debugging machine motion from the wrong program.

The October 25 printer preview exposed the layer count, G-code line count and a warning about priming moves. That warning is one of the details I value most in the late prototype. If a movement was omitted from the preview, the interface gave us a place to start looking for the reason. To compare the preview with the simulated printer, we would still need the same file revision and layer. To say what happened to a physical part, we would also need the assignment and completion record for that run. The journal stops at the inspection stage, with no measured end-to-end production result to report.

The wider project touched NVIDIA Isaac Kit, MATLAB, National Instruments tooling, SQL, retrieval-augmented generation, agentic workflows, kernel-level development, Win32 packaging, Electron and embedded Python. What I remember most clearly is the moment a large problem became small enough to test. Follow the nozzle through its parent frame. Subscribe a viewer after the first event. Ask which attempt a timestamp belongs to. Each took us far enough to make the next engineering decision without guessing what had happened in between.

## Prototype release

We plan to make the prototype public once the patent is granted.
