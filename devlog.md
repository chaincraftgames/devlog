## Before Jan 30, 2025 ##
Initial focus was on creating a gamedef and validation that we could use to inform the AI if the gamedef was valid.  The idea was the gamedef would be used to describe the game in sufficient details to allow for the game code to be generated.  Gamedef is yaml files + lua script.

Created a game design agent that would converse with a user to produce a complete specification that could be used by a set of agents to produce a gamedef.

Integrated design agent with Discord so our discord members could interact with it and start producing designs that we could use as test data for game builder agents.

After discord bot work, went back to creating agents that could produce gamedef.  Started with a prescriptive approach where we told the agents exactly what steps to take when.  One day I had a brainstorm for an emergent approach where we provide high level goals to planner agents and then task them with creating instructions for other agents to produce the gamedef.  Shifted to the emergent approach, focussing on planner agents.  Discovered that agents produce better responses when I split them into a reasoning step and a formatting step.  Trying to produce formatted results using a new structure that LLM wasn't trained on produced suboptimal results.  This in turn introduced a problem when either the reasoning step or the formatting step introduced an error.  I had an error correction loop, but if the formatter produced the error the reasoning agent didn't necessarily need corrective action and in fact doing so could be detrimental.   I developed an approach for a a self healing network of agents where when one of the agents makes a mistake, another agent gets analyzes all of the agent inputs and outputs and determines the corrective action and which agent made the error and routes the execution to the agent with the corrective action.  From there the network processes as before but hopefully without the original mistake.  Completed the happy path development (no errors) and began work on introducing error correction. 

Pivoted in January to working on a simple text based engine and a rock, paper, scissors reference implementation for games that could be played on Discord or Telegram.  The reasons were to get some quick wins and demonstrate that AI could create and modify games using the engine.  I believe that we may learn through this process that a hybrid of some yaml config plus AI generated code could be optimal for our gamebuilder agents.  We will certainly learn a lot through trying this approach and I am grateful for the pivot.

## January 30, 2025 ##
Got the text game engine and rps module working to play multiple complete games.  Found a bug when a player plays the same piece in the 1st and 3rd rounds.  

## Feb 1, 2025 ##
Fixed the bugs with playing the same piece for the 1st and 3rd piece.  The issue was a redundant addComponent in the input handler for rock.  Started refactor of state system configuration to simplify for AI creation

## Feb 2, 2025 ##
Continued work on state system refactor.  Multiple iterations of the format.  Working to support nested rounds.  Even though it isn't required for RPS, it will be required for more complex games so it will be needed for AI modifications.

## Feb 3, 2025 ##
Continued work on state system refactor.  Scrapped the current version in an attempt to simplify config again.  Much happier with config now, and getting close to tests passing.

## Feb 5, 2025 ##
Continued work on state system refactor.  Had idea to map transitions config into a set of generators.  Simplified iteration, state management, etc...  AI keeps getting confused and mixing creation time and runtime considerations.  

## Feb 8, 2025 ## 
Continued work on state system refactor.  Refactored transitions config so that repeat transitions have a target state and we increment iteration or check condition when target state is reached.  This solves the problem of when to increment because we didn't want to require every transition in a repeat to be triggered; transitions could take different paths to get to the target state and we should choose the transition based on the one that matches the triggering condition.  Implemented tests for simple non-repeat cases and they passed.
  
Thought - We can have an agent responsible just for creating the state config for the game.  It can stub out the actions and conditions and another agent can implement them.  This will allow us to narrow down the context and focus the state config gen agent.

## Feb 9, 2025 ##
Fixed all known issues with the GameStateSystem.  All tests pass.  Updated RPS module to use the new config and it gets stuck waiting for player input.  The issue is that we need to execute the first transition in the repeat in order to start the round and make the player active so they can submit input.  Realized the whole active thing is problematic and needs to be reworked so I commented out for now.  That still leaves a problem of the player won't get prompted until the startRound occurs.  I decided to add an ability for a repeate to have an execute that is executed when the repeat is triggered.  Created a test to verify.  The test exposed an issue where repeats that consist of a single transition wil always be triggered before the transition that follows them.  The problem is that if the repeat condition is a simple () => true we can't tell wether the repeat has completed or not.  A solution is to require that repeat always have 2 states.  A better solution is to have a condition that can check for a repeat to be complete.  

## Feb 10, 20025 ##
Discovered another issue in testing.  We aren't advancing the state every time we should.



