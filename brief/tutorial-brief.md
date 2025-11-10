# Brief: Simple 2D Platformer Tutorial for Godot

## Project Overview

Create a concise, beginner-friendly tutorial that teaches readers how to build a minimal 2D platformer in Godot. This tutorial should be significantly shorter and more focused than the official "Your First 2D Game" tutorial, covering only the essential mechanics needed for a basic platformer.

## Target Audience

- Beginners who are new to Godot
- Developers familiar with other game engines looking to learn Godot quickly
- Anyone wanting to understand basic 2D platformer mechanics

## Objectives

Create a working 2D platformer with the following features:
1. A playable character that can be controlled with arrow keys
2. A floor/ground that the character stands on (no falling through)
3. Basic jump functionality
4. Screen boundaries to keep the character visible

## Deliverables

A complete tutorial that includes:
- Project setup instructions
- Step-by-step guidance for creating the player character
- Implementation of keyboard controls (arrow keys for movement)
- Adding a static floor/platform
- Implementing jump mechanics
- Working code examples for each feature
- Clear explanations of how the code works

## Technical Requirements

- Use Godot 4.x
- Write code in GDScript
- Keep the tutorial focused - aim for completion in under 30 minutes
- Provide complete, working code snippets
- Explain key Godot concepts as they're introduced (nodes, scenes, physics, input)

## Scope Limitations

This tutorial should NOT include:
- Enemy characters or combat
- Score system or HUD
- Multiple levels
- Advanced animations
- Sound effects or music
- Collectible items
- Complex physics interactions

## Success Criteria

By the end of this tutorial, readers should have:
- A functional 2D platformer they can play
- Understanding of basic Godot node structure
- Knowledge of how to handle player input
- Basic grasp of 2D physics in Godot
- Confidence to expand the project with their own features

## Style and Tone

- Write in an encouraging, accessible tone
- Use "we" voice (e.g., "we'll create a player character")
- Avoid unnecessary jargon
- Explain technical terms when first introduced
- Focus on practical implementation over theory
- Provide context for why we're doing each step

## Key Concepts to Cover

- Scenes and nodes
- CharacterBody2D for the player
- StaticBody2D for platforms
- Input handling with Input.is_action_pressed()
- Basic physics (gravity, velocity, jump)
- The _physics_process() function
- Collision shapes and layers
