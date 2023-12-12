# APEX Graph From Templates
A template based alternative for creating custom APEX rigs in Houdini
### BETA!

## Installation
Copy the HDA into your Houdini install's OTLS folder. Requires Houdini 20.

## Use
### See the example HIP for basic usage. ###

- First input - an existing APEX graph (optional)
- Second input - skeleton
- Third input - packed APEX templates

## What does it do?
It resolves names in the template graphs based on a "context" attribute on the skeleton, then prints and connects those nodes in a new graph.

For example if your skeleton has 3 joints named "shoulder", "elbow" and "wrist", a node in your APEX template called "\_\_\_NAME\_\_\_" will be printed three times, each time replacing "\_\_\_NAME\_\_\_" with the skeleton joint's name. (Text to replace is surrounded by triple underscores "___")

If the APEX nodes in your template are linked to another node called "Transform___NAME___", three more nodes will be printed ("Transform_shoulder", "Transform_elbow" and "Transform_wrist"), and each linked to its matching node for that point.

You can also use template replacement in APEX node inputs/outputs and string parameters.

## How to define what strings to replace?
The replacements are defined by a dictionary attribute called "context" on your skeleton. The node can automatically create some based on KineFX relationships (eg NAME, PARENT, FIRSTCHILD), you can add some as KEY:value pairs in the SOP node's parameters, or you can add a detail or point attribute directly to your skeleton, or a mix of all these.

## How does it work?
- The context attribute for each point is fed into the selected APEX templates to replace the names in those templates, then nodes are printed and connected based on the resolved names.
- The node will run over each point to resolve APEX names, but will not necessarily add new nodes for each point - existing nodes (nodes with the same name) will not be replaced.
   - For example if you have an IK setup with a context pair LIMB:Left_arm that's shared by the shoulder, elbow and wrist of that arm, a node called "Transform___LIMB___" will only be printed once for those points and can be shared by all of them.
   - Templates can also be layered, so if one template contains the APEX node "\_\_\_NAME\_\_\_", a later template applied to the same point that also contains a node named "\_\_\_NAME\_\_\_" will not print a new node, but all new links will be applied to the already existing node. Links are only added or replaced - never removed - so you can create templates that refer to existing nodes without having to recreate all their inputs in each template.

## Advanced
The following features aren't well tested yet, beware...
- Switch nodes - you can follow a different input path depending on a value from your context. A switch node is a null whose name starts with "\_\_\_SWITCH\_" followed by a key from your context dictionary. The input path chosen depends on the value of that key. __Important gotcha:__ the template system finds ports in alphabetical order, no matter which order they originally appear on your template nodes - this works with the default null port namings ("\_\_spare\_\_0", "\_\_spare\_\_1" etc), but watch out if you change these, the switch node might not select the path you expect if they aren't in alphabetical order.
- Loop nodes - a section of the APEX graph can be repeated multiple times, optionally with feedback. Start the loop with a null node whose name starts with "\_\_\_FORBEGIN" (input is always as set in the graph)  or "\_\_\_FORBEGIN_FEEDBACK" (input for first iteration is as set in the graph, afterwards the input is set to the end of the previous loop). End the loop with a null whose name starts with "\_\_\_FOREND\_" followed by a key from your context dictionary - the value of the key determines the number of loops. Include "\_\_\_LOOP_ITERATION\_\_\_" somewhere in the name of nodes you want to print multiple times with the loop. The for loop also adds these context keys: LOOP_NEXT, LOOP_PREV, LOOP_FIRST, LOOP_LAST (replace LOOP_ITERATION with one of these to match the same node name in a different loop), LOOP_ISFIRST and LOOP_ISLAST.
- TODO: examples for these.
 
## Why?
I've created APEX Graph from Templates as an alternative to create custom APEX graphs, as I wasn't totally comfortable with the autorig components that need to know themselves how to edit the graph they're applied to. In this system each template is closer to the end result, but with some helpful automation and layering so you can still reuse templates and don't have to repeat yourself too much for dense skeletons.

The system is inspired by my time spent coding templates for dynamic websites with Django, you don't have to know how that works but if you have any experience with those then the reasoning behind this might make more sense.

## Cons?
- It's Python based so much slower to generate graphs than the autorig components made with APEX, especially for detailed templates with lots of connections.
- Only string parameters can be edited by the template, all other values are left as is (so numbers can't vary per point, eg in a TransformObject's localtransform attribute).
- It's still an abstraction from the end result, you might find it more or less comfortable working this way.
- A very heavy BETA warning, it may break and it might not give useful error messages when it does.
### Known issues
- Nodes within subgraphs need to have unique names. The code won't replace anything inside a subgraph (this is by design - and you can put heavy sections in subgraphs to speed things up a lot, since it just prints the whole subgraph in one without having to calculate the links), but those nodes still confuse the code if they have the same name - sadly this includes the "parms" and "output" node so you have to be careful with subgraphs.
