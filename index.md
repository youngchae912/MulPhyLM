---
layout: project_page
permalink: /

title: "MulPlanLM: multimodal robotic task planning with vision-language models and physical feedback"
authors:
  - '<a href="https://youngchae912.github.io/">Young-Chae Son</a>, Dong-Han Lee and Soo-Chul Lim*'
affiliations:
  - Department of Mechanical, Robotics and Energy Engineering, Dongguk University, Seoul 04620, South Korea

---

<style>
table.paper-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.92rem;
}
table.paper-table th,
table.paper-table td {
  text-align: center;
  padding: 0.45rem 0.5rem;
  border: none;
}
table.paper-table thead tr:last-child th {
  border-bottom: 1px solid #363636;
}
table.paper-table thead tr:first-child th {
  border-top: 1.5px solid #363636;
}
table.paper-table tbody tr:last-child td {
  border-bottom: 1.5px solid #363636;
}
table.paper-table td.rowhead,
table.paper-table th.rowhead {
  text-align: left;
  font-weight: 600;
}
table.paper-table tr.avg td {
  border-top: 1px solid #363636;
  font-weight: 600;
}
table.paper-table tr.group td {
  border-top: 1px solid #363636;
}
.table-wrap {
  overflow-x: auto;
}
</style>

<div class="has-text-centered" style="margin-top: -3rem; margin-bottom: 2rem;">
  <p class="is-size-5" style="font-weight: 600;">
    Our research has been accepted for publication in Intelligent Service Robotics, 2026.
  </p>
</div>

<!-- Using HTML to center the abstract -->
<div class="columns is-centered has-text-centered">
    <div class="column is-four-fifths">
        <h2>Abstract</h2>
        <div class="content has-text-justified">
This study proposes a multimodal robotic task-planning framework based on large language models (LLMs) that utilizes both visual and force-derived physical feedback. The system integrates multimodal inputs, including camera images and force/torque (F/T) sensor data, to interpret the visual and physical properties of objects and generate a task sequence to execute given commands. The integration of vision and force data allows the system to compensate for the limitations of each modality, leading to more reliable decision-making. The framework consists of three components: Extractor, Planner, and Sub-planner. According to their assigned roles, each agent automatically translates high-level commands given in natural language into executable robot motion plans, enabling the robot to perform the required sequence of actions. The system combines visual and force data to handle tasks that are difficult or infeasible with a single modality. Furthermore, it supports sequential and condition-based task planning through the collaboration of multiple LLM agents. Experimental results in various task scenarios show that the proposed framework consistently improves overall task success rates compared with unimodal settings with different LLMs and achieves a higher success rate compared to using only visual or force data.
        </div>
    </div>
</div>

---

## Motivation

<div class="content has-text-justified">
<p>
Traditional industrial automation depends on rule-based scripts or domain-specific programming languages, so every setup or reconfiguration demands skilled engineers and long development cycles. LLMs can translate natural language instructions into executable code, control scripts, or robot commands, which shifts task instruction away from expert-driven workflows and lets non-experts interact with robots.
</p>
<p>
But executing a command in the physical world requires grounding it in sensory data. Most existing VLM-based approaches reason over RGB images alone and never explicitly represent <strong>non-visible physical state</strong>: the contents of an opaque container, the weight of an object, the contact forces during manipulation. Frameworks that do add tactile or audio signals typically treat them as auxiliary channels rather than integrating them into the reasoning process itself.
</p>
<p>
<strong>MulPlanLM</strong> addresses this gap. It combines spatial understanding from RGB images with force-derived physical feedback from a wrist-mounted F/T sensor, and processes both through a hierarchical three-agent architecture — Extractor, Planner, and Sub-Planner — so that high-level planning and low-level code generation are both grounded in visual <em>and</em> physical context.
</p>
</div>

### Contributions

- **Reasoning in partially observable environments.** Vision-only baselines fail when the task-relevant state is not visible, such as determining the contents of an opaque container. Integrating force-derived feedback with visual input lets the robot infer non-visible states and plan under partial observability.
- **Hierarchical LLM-based task planning.** A three-layer agent structure translates abstract natural language commands into precise robot actions by leveraging integrated sensory context, with a verifiable intermediate plan between perception and code.
- **Evaluation across representative models.** MulPlanLM is evaluated on 15 real-world tasks with GPT-4V + GPT-4.1, Gemini 2.5 Pro, and LLaMA 4 Maverick, improving task success across all three — especially where visual information alone is insufficient.

---

## Framework

![Framework](/static/image/figure1.png)

<div class="content has-text-justified" style="margin-top: 10px;">
<p>
<strong>Figure 1:</strong> (a) The user interface, through which a natural-language <em>Task Goal</em> is submitted and a real-time camera feed is monitored. (b) The system architecture: three LLM-based agents with a clear separation of concerns. The <strong>Extractor</strong> analyzes the environment from RGB images and end-effector force measurements; the <strong>Planner</strong> designs a high-level task plan from that analysis; the <strong>Sub-Planner</strong> translates the plan into executable robot code, with a retry mechanism that regenerates failed code up to three times.
</p>
</div>

![Data flow](/static/image/scheme.png)

<div class="content has-text-justified" style="margin-top: 10px;">
<p>
<strong>Figure 2:</strong> Data flow from multimodal perception to action execution. Each modality — the natural-language goal, RGB-based visual detections, and force/torque signals — is processed and structured before being integrated into the Extractor input space, then passed through the Planner and Sub-Planner reasoning pipeline.
</p>
</div>

![Agent inputs and outputs](/static/image/method.png)

<div class="content has-text-justified" style="margin-top: 10px;">
<p>
<strong>Figure 3:</strong> Input and output structure of each agent. All three receive the same task goal, and the prompts stay consistent regardless of the task. (a) The Extractor analyzes the environment using vision and force sensor data. (b) The Planner generates a high-level task plan from the task goal and environmental information. (c) The Sub-Planner converts that plan into executable robot control code.
</p>
</div>

### Extractor

<div class="content has-text-justified">
<p>
The Extractor perceives and interprets the environment, emitting structured environmental information from sensor inputs and user instructions. Its multimodal input is the set $X_t = \{l, I_t, \bar{F}, \mathcal{G}_{ext}\}$:
</p>
<ul>
  <li>$l$ — the natural-language <em>Task Goal</em> provided by the user.</li>
  <li>$I_t$ — the RGB image annotated with object detection results, yielding an object set $O_t = \{(b_i, c_i)\}_{i=1}^{N}$, with bounding box $b_i$ and class ID $c_i$ superimposed on the raw image.</li>
  <li>$\bar{F}$ — force feedback from the wrist-mounted 6-DoF F/T sensor on the UR5e. The raw measurement $F_t = [F_x, F_y, F_z]^T \in \mathbb{R}^3$ is collected at 100 Hz for 1 second after grasping, and the averaged vector $\bar{F}$ serves as the final input.</li>
  <li>$\mathcal{G}_{ext}$ — guidelines specifying how to interpret multimodal correlations and handle missing sensor data.</li>
</ul>
<p>
The averaged force vector is serialized into structured numerical text tokens and appended to the VLM's textual input. From these inputs the Extractor infers semantic context about the target object and its surroundings without task-specific prior knowledge, and emits structured JSON in two segments: <em>Visible Object Information</em> (object ID, shape, size, and the estimated mass $\hat{m}$ derived from $\bar{F}$) and <em>Visible Environment Information</em> (element types, appearance, spatial locations). Because the output is structured text rather than a model-specific representation, the framework is not tied to any one VLM architecture, and the reasoning process stays interpretable.
</p>
</div>

### Planner

<div class="content has-text-justified">
<p>
The Planner generates a high-level natural-language task plan from the Extractor's analysis. Prior work often generates entire plans or control code in a single end-to-end pass, which can produce execution errors by omitting a verifiable intermediate reasoning step or failing to incorporate environmental feedback. MulPlanLM instead introduces an explicit intermediate planning layer: a human-readable plan that can be verified and modified, and into which contextual information is integrated, before any code is generated.
</p>
<p>
It takes three inputs — the user's Task Goal; output guidelines comprising an <em>Action Description</em> of basic robot capabilities, <em>Output Examples</em> demonstrating possible action combinations, and <em>High-Level Task Constraints</em> guiding step-by-step composition; and the Extractor's environmental analysis. Formally, given the structured environment representation $\mathcal{E}_t$, it produces the plan $P_t = \mathrm{Planner}(\mathcal{E}_t)$.
</p>
</div>

### Sub-Planner

<div class="content has-text-justified">
<p>
The Sub-Planner interprets the natural-language plan and converts it into executable Python control code composed of predefined API-level actions. It takes five inputs: the Task Goal; output guidelines (<em>Action Description</em>, <em>Output Examples</em>, and <em>Low-Level Task Constraints</em> describing motion limits and axis-based rules); the Extractor output; the Planner output; and the object position list from the RGB-D camera.
</p>
<p>
The action space $\mathcal{A}$ is a finite set of primitives, restricted to four API calls:
</p>
<ul>
  <li><code>move_robot_to_tcp(target_pose)</code></li>
  <li><code>get_robot_tcp()</code></li>
  <li><code>gripper(command)</code></li>
  <li><code>check_visual_condition()</code></li>
</ul>
<p>
Generated code is statically verified before execution. Code containing unauthorized commands or invalid formatting is regenerated up to three times before the trial is aborted, and even valid API calls are rejected if they violate workspace boundaries or safety constraints. This dual validation at the API and controller levels keeps execution safe and bounded.
</p>
</div>

---

## Experimental Setup

![Experimental setup](/static/image/figure9.png)

<div class="content has-text-justified" style="margin-top: 10px;">
<p>
<strong>Figure 4:</strong> Overall experimental setup. Object positions were obtained from a RealSense D435 camera, while images from a RealSense L515 camera were used as input to the Extractor.
</p>
</div>

<div class="content has-text-justified">
<p>
The platform consists of a Universal Robots UR5e arm with a built-in F/T sensor, an OnRobot 2FG7 parallel gripper, and two Intel RealSense RGB-D cameras (L515 and D435). An object detector provides bounding boxes for object detection and position estimation.
</p>
<p>
Detection was reliable in this setup, so the performance differences analyzed below are more likely attributable to multimodal reasoning, force-based state interpretation, and code-level conditional execution than to detection failures.
</p>
<p>
Three LLM configurations were evaluated: OpenAI GPT-4V + GPT-4.1 (GPT-4V as the visual information extractor, GPT-4.1 for the planner and sub-planner), Google Gemini 2.5 Pro, and Meta LLaMA 4 Maverick.
</p>
</div>

### Task Design

<div class="content has-text-justified">
<p>
15 tasks were designed against two criteria: <strong>modality coverage and discriminability</strong> across input conditions, and <strong>progressive reasoning complexity</strong>. Each task is scored as a binary success or failure on completion of a predefined mission, with five repeated trials per model and input condition.
</p>
<p>
The tasks are split evenly into three categories of five. Vision-based tasks turn on object shape and size (Task 1), color (Tasks 3–4), and other distinguishing features (Tasks 2, 5). Force-based tasks reason about object weight (Tasks 6–10) — in Task 8, for instance, the target moves out of the camera's field of view when pushed, which removes visual comparison entirely and strictly requires force-based state reasoning. Multimodal tasks (Tasks 11–15) need both at once: reading text on paper while weighing an object, comparing weights of visually identified blocks, or selecting the lightest of several filled containers.
</p>
<p>
Complexity increases across the set. Some tasks are single-attribute selection; Task 8 requires detecting weight changes at intermediate steps; Tasks 9 and 10 require distinct choices under specific weight conditions; and the multimodal tasks require conditional action planning over synthesized multimodal context. Execution times estimated from the recorded videos follow the same gradient — roughly 15 s for vision-based pick-and-place, 30–40 s for force-based tasks, and about 1 min for multimodal tasks. These durations come from video observation rather than instrumented logging and are therefore approximate.
</p>
</div>

<div class="table-wrap">
<table class="paper-table">
  <thead>
    <tr>
      <th class="rowhead">Type</th>
      <th>Task</th>
      <th class="rowhead">Task Description</th>
    </tr>
  </thead>
  <tbody>
    <tr><td class="rowhead" rowspan="5">Vision-Based</td><td>1</td><td class="rowhead">Hand over the smallest square-shaped object in sight.</td></tr>
    <tr><td>2</td><td class="rowhead">Lift the box that contains the same object as the one placed on top of the wooden box.</td></tr>
    <tr><td>3</td><td class="rowhead">Pick up the fruit that has the most similar color to the floor.</td></tr>
    <tr><td>4</td><td class="rowhead">Pick up the fruit that has the most different color from the floor.</td></tr>
    <tr><td>5</td><td class="rowhead">Hand over a ball suitable for playing golf.</td></tr>
    <tr class="group"><td class="rowhead" rowspan="5">Force-Based</td><td>6</td><td class="rowhead">If a container has 15 or more metal balls inside, leave it in place. Otherwise, move it to the right.</td></tr>
    <tr><td>7</td><td class="rowhead">If the Eclipse container has contents remaining, hand it over. Otherwise, return it to its original position.</td></tr>
    <tr><td>8</td><td class="rowhead">Move the black box to the right. If any blocks inside are missing, return it to its original position.</td></tr>
    <tr><td>9</td><td class="rowhead">If the black box contains more than five blocks, hand it over. Otherwise, leave it in place.</td></tr>
    <tr><td>10</td><td class="rowhead">Among two blocks, select and hand over the heavier one.</td></tr>
    <tr class="group"><td class="rowhead" rowspan="5">Multimodal (V+F)</td><td>11</td><td class="rowhead">Among the papers, place the container in the correct location.</td></tr>
    <tr><td>12</td><td class="rowhead">Among the papers, place the box in the correct location.</td></tr>
    <tr><td>13</td><td class="rowhead">Among various objects, find the blocks and lift the lightest one.</td></tr>
    <tr><td>14</td><td class="rowhead">If a red block is present, measure its weight and check if it is over 0.1 kg. If it is over 0.2 kg, move it forward. If not, return it to its original position. If a blue block is present, move it forward regardless of its weight.</td></tr>
    <tr><td>15</td><td class="rowhead">Pick up each container filled with bolts to assess the weight, then hand over the one that is less heavy.</td></tr>
  </tbody>
</table>
</div>
<p class="has-text-centered is-size-7 mt-2">
  <strong>Table 1:</strong> Tasks categorized by the importance of vision and force sensor information.
</p>

---

## Results

<div class="table-wrap">
<table class="paper-table">
  <thead>
    <tr>
      <th class="rowhead">Task Type (Range)</th>
      <th class="rowhead">Model</th>
      <th>V+F (%)</th>
      <th>Vision (%)</th>
      <th>Force (%)</th>
    </tr>
  </thead>
  <tbody>
    <tr><td class="rowhead" rowspan="4">Vision-based (1–5)</td><td class="rowhead">GPT-4V + GPT-4.1</td><td>84</td><td>76</td><td>12</td></tr>
    <tr><td class="rowhead">Gemini 2.5 Pro</td><td>80</td><td>80</td><td>4</td></tr>
    <tr><td class="rowhead">LLaMA 4</td><td>76</td><td>80</td><td>12</td></tr>
    <tr class="avg"><td class="rowhead">Average</td><td>80.0 ± 9.3</td><td>78.7 ± 9.1</td><td>9.3 ± 7.8</td></tr>
    <tr class="group"><td class="rowhead" rowspan="4">Force-based (6–10)</td><td class="rowhead">GPT-4V + GPT-4.1</td><td>72</td><td>20</td><td>36</td></tr>
    <tr><td class="rowhead">Gemini 2.5 Pro</td><td>76</td><td>32</td><td>48</td></tr>
    <tr><td class="rowhead">LLaMA 4</td><td>68</td><td>24</td><td>36</td></tr>
    <tr class="avg"><td class="rowhead">Average</td><td>72.0 ± 9.6</td><td>25.3 ± 12.4</td><td>40.0 ± 7.8</td></tr>
    <tr class="group"><td class="rowhead" rowspan="4">Multimodal (11–15)</td><td class="rowhead">GPT-4V + GPT-4.1</td><td>68</td><td>40</td><td>4</td></tr>
    <tr><td class="rowhead">Gemini 2.5 Pro</td><td>80</td><td>52</td><td>4</td></tr>
    <tr><td class="rowhead">LLaMA 4</td><td>60</td><td>32</td><td>0</td></tr>
    <tr class="avg"><td class="rowhead">Average</td><td>69.3 ± 12.4</td><td>41.3 ± 9.6</td><td>2.7 ± 4.5</td></tr>
  </tbody>
</table>
</div>
<p class="has-text-centered is-size-7 mt-2">
  <strong>Table 2:</strong> Success rates across task types under different input modality conditions. Five tasks per category, five repeated trials per model and input condition.
</p>

### Vision-based tasks (1–5)

<div class="content has-text-justified">
<p>
Every model performs well whenever visual information is present and collapses without it. GPT-4V + GPT-4.1 reaches 84% with vision and force against 76% vision-only, but falls to 12% on force alone. Gemini 2.5 Pro and LLaMA 4 follow the same pattern — 80% in both the multimodal and vision-only settings, dropping to 4% and 12% respectively on force alone. Vision is the primary cue for this group, and adding force provides limited additional benefit.
</p>
</div>

### Force-based tasks (6–10)

<div class="content has-text-justified">
<p>
Here the ordering inverts. Vision-only success is low across all three models, since the state of stacked or occluded objects cannot be inferred visually. Force alone solves some tasks by combining positional and force cues, but without visual context it still trails the multimodal condition. Task 10 is the hardest in this group — the weight difference between objects is minimal, which makes force cues alone difficult to discriminate.
</p>
<p>
Gemini 2.5 Pro achieves the highest average here. GPT-4V + GPT-4.1 is slightly lower overall but consistent across tasks, and LLaMA 4 is likewise stable in force-dominant settings. Force alone enables moderate performance; combining it with vision improves it further.
</p>
</div>

### Multimodal tasks (11–15)

![Multimodal example](/static/image/figure3_1.png)

<div class="content has-text-justified" style="margin-top: 10px;">
<p>
<strong>Figure 5:</strong> Success and failure cases for Only Force, Only Vision, and the proposed method given the same task goal. Because the box is actually empty, it should be moved onto the paper labeled "Empty."
</p>
</div>

<div class="content has-text-justified">
<p>
These tasks are decision problems that need vision and force jointly — appearance alone is insufficient, and force alone cannot supply a complete interpretation. Tasks 11 and 12 score relatively high across all models: they analyze force information against visual cues written on paper, so the two modalities interact without conflict and their complementary effect shows clearly.
</p>
<p>
For every model the best result comes from using both modalities. Performance drops sharply under vision-only or force-only conditions because essential cues are missing — force-only success in this group is near zero (0–4%). Both modalities are necessary for reliable performance here.
</p>
</div>

### Overall

![Overall task performance](/static/image/figure7.png)

<div class="content has-text-justified" style="margin-top: 10px;">
<p>
<strong>Figure 6:</strong> Success rates (%) for overall task performance (Tasks 1–15) across GPT-4V + GPT-4.1, Gemini 2.5 Pro, and LLaMA 4 under three input modalities: Vision + Force, Vision only, and Force only. Error bars represent standard error across trials.
</p>
</div>

<div class="content has-text-justified">
<p>
Averaged over all 15 tasks, the multimodal condition is both the highest and the most consistent. Gemini 2.5 Pro shows the highest overall consistency across tasks, GPT-4V + GPT-4.1 performs well where visual interpretation dominates, and LLaMA 4 stays relatively stable where force-based reasoning dominates.
</p>
<p>
This matters for deployment. Vision-only approaches work when clear visual cues are available but fail under internal-state inference or occlusion — conditions that are common in practical robotic environments. Force data acts as the complementary source that covers those cases.
</p>
<p>
Breaking the results down by category exposes the framework's scope and its systematic limits. Vision-based tasks average <strong>80.0% (±9.3)</strong> and force-based tasks <strong>72.0% (±9.6)</strong>; both involve mostly single-modality reasoning and threshold-based physical judgments, where performance is relatively stable. Tasks requiring genuine integration of vision and force average <strong>69.3% (±12.4)</strong> — lower, though still a clear advantage over the corresponding unimodal baselines (41.3% vision-only, 2.7% force-only).
</p>
<p>
Degradation concentrates in tasks that require (1) intermediate state updates during execution, (2) deep conditional branching, or (3) sequential comparative reasoning across multiple objects. Failures in these cases typically came from incomplete condition updates at the code level rather than from misreading the force signals — the framework is stable on single-step perceptual or threshold-based reasoning, while explicit intermediate state tracking and hierarchical procedural reasoning would benefit from further refinement.
</p>
</div>

---

## Real-World Execution

<section class="hero teaser" style="padding: 0;">
  <div class="container is-max-desktop" style="padding: 0;">
    <div class="hero-body" style="padding: 0; position: relative;">
      <video 
        id="teaser" 
        autoplay 
        muted 
        loop 
        playsinline 
        style="width: 100%; height: auto; display: block; object-fit: cover;">
        <source src="static/image/testvideo.mp4" type="video/mp4">
      </video>
    </div>
  </div>
</section>

<section class="hero teaser" style="padding: 0;">
  <div class="container is-max-desktop" style="padding: 0;">
    <div class="hero-body" style="padding: 0; position: relative;">
      <video 
        id="teaser" 
        autoplay 
        muted 
        loop 
        playsinline 
        style="width: 100%; height: auto; display: block; object-fit: cover;">
        <source src="static/image/testvideo2.mp4" type="video/mp4">
      </video>
    </div>
  </div>
</section>

---

## Discussion

<div class="content has-text-justified">
<p>
Integrating visual information with force-derived physical feedback improves LLM-based task planning specifically when task-relevant physical states are not directly observable from vision. Vision supplies symbolic and spatial grounding for identifying and localizing candidate objects; force supplies the additional evidence that resolves hidden properties such as weight or internal contents. The improvement therefore comes from <em>modality complementarity</em> rather than redundant sensing.
</p>
<p>
The framework does not attempt to solve low-level contact-rich manipulation, continuous force regulation, or force-control policy learning. Its contribution is enabling force-based state reasoning within a symbolic task-planning framework, not addressing contact dynamics directly.
</p>
<p>
Model-dependent performance differences further suggest that reasoning over force-derived numerical signals may be partially separable from visual scene understanding. These observations are specific to this framework and task setup, and should not be read as a general benchmark of model-level performance.
</p>
<p>
Failure analysis points at downstream reasoning and planning rather than object-level perception. Smaller weight differences and visual occlusion did appear to increase difficulty, but they were not independently controlled as a predefined difficulty axis. Future work will parameterize task difficulty by systematically varying occlusion level, lighting condition, contact ambiguity, weight-gap granularity, and conditional branching complexity. Task completion time also warrants separate treatment: timing statistics were not systematically recorded across all trials, so future evaluations will measure VLM/LLM inference time, planning time, code generation time, physical execution time, and recovery procedures independently.
</p>
</div>

## Conclusion

<div class="content has-text-justified">
<p>
MulPlanLM is an LLM-based robotic task-planning framework that integrates visual information and force-derived physical feedback for physically grounded decision-making. It improves task execution where vision or force alone provides incomplete information, particularly when hidden physical states such as object weight or internal contents must be considered. Incorporating explicit state tracking, post-condition verification, and closed-loop replanning is expected to further improve performance on tasks involving conditional reasoning and sequential decision-making.
</p>
</div>

---

## Citation

<div class="box" style="background-color: #fafafa; border: 1px solid #e8e8e8;">
<pre style="white-space: pre-wrap; word-break: break-word; font-size: 0.95rem;"><code>@article{son2026mulplanlm,
  author  = {Son, Young-Chae and Lee, Dong-Han and Lim, Soo-Chul},
  title   = {MulPlanLM: Multimodal Robotic Task Planning with Vision-Language Models and Physical Feedback},
  journal = {Intelligent Service Robotics},
  year    = {2026}
}
</code></pre>
</div>

<div class="content has-text-justified is-size-7" style="color: #666;">
<p>
This work was supported by the National Research Foundation of Korea (NRF) grant funded by the Korea government (MSIT) (RS-2025-00562981, RS-2025-25433409).
</p>
</div>
