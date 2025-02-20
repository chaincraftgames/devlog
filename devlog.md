## Before Jan 30, 2025 ##
Initial focus was on creating a gamedef and validation that we could use to inform the AI if the gamedef was valid.  The idea was the gamedef would be used to describe the game in sufficient details to allow for the game code to be generated.  Gamedef is yaml files + lua script.

Created a game design agent that would converse with a user to produce a complete specification that could be used by a set of agents to produce a gamedef.

Integrated design agent with Discord so our discord members could interact with it and start producing designs that we could use as test data for game builder agents.

After discord bot work, went back to creating agents that could produce gamedef.  Started with a prescriptive approach where we told the agents exactly what steps to take when.  One day I had a brainstorm for an emergent approach where we provide high level goals to planner agents and then task them with creating instructions for other agents to produce the gamedef.  Shifted to the emergent approach, focussing on planner agents.  Discovered that agents produce better responses when I split them into a reasoning step and a formatting step.  Trying to produce formatted results using a new structure that LLM wasn't trained on produced suboptimal results.  This in turn introduced a problem when either the reasoning step or the formatting step introduced an error.  I had an error correction loop, but if the formatter produced the error the reasoning agent didn't necessarily need corrective action and in fact doing so could be detrimental.   I developed an approach for a a self healing network of agents where when one of the agents makes a mistake, another agent gets analyzes all of the agent inputs and outputs and determines the corrective action and which agent made the error and routes the execution to the agent with the corrective action.  From there the network processes as before but hopefully without the original mistake.  Completed the happy path development (no errors) and began work on introducing error correction. 

Took a slight detour to support the minting of our Data Drifters collection.  Used ArtEngine to generate json metadata files with traits and wrote scripts that use Claude to generate MJ prompts based on the metadata, and deploy to IPFS.

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

## Feb 10, 2025 ##
Discovered another issue in testing.  We aren't advancing the state every time we should.  Determined that the issue is caused by using a generator for the transitions provider so the generator was being reentered in a place where it couldn't process the next transition, requiring another tick tio trigger the next transition.  The fix was to change transitions provider to a normal function so we always ran the search over all transitions each tick.  

## Feb 11, 2025 ##
After fixing the issue with transitions, discovered the nested repeat case is failing because the inner repeat is being being reset continuously after completing the outer repeat completes one iteration.  

## Feb 12, 2025 ##
Spent some time trying to fix nested repeat, but ultimately was not successful in resolving the issue.  Shifted gears to getting everything delivered, producing documentation to allow AIs to generate game modules.  Produced a video explaining how to pull down, build the code, and run the example.

## Feb 13, 2025 ##
Forgot that we were using a local bitECS.  In the course of updating the video to show how to use a fresh bitECS, discovered that the latest rc-0-4-0 broke something in the engine.  Have not had time to dig in and determine cause.

## Feb 14, 2025 ##
During a team discussion, we determined that there is value in having an ability to simulate gameplay using an AI.  The value we see is:
* Allows us to protoitype the end to end experience of user creating game, remixing game, playing games.
* Audience building and community engagement.
* Prototype PAIT concept
* Might make it easier to get grants
* Simulated gameplay could be used during game design.  Could become the conversational artifact.
we decided to do a timeboxed experiment to see if we could get an LLM to be the GameModule implementation for the text-game-engine.

## Feb 16, 2025 ##
Began working on integrating AI simulation into text-game-engine.  Decided to use an LLM to extract a game state object from the game description.  We'll use the AI response to generate a zod schema that will be used for runtime output parsing.

## Feb 17, 2025 ##
Continue with AI integration.  Created a function to process the game description and generate a schema that can be used to parse and validate the runtime state.  Initially struggled with getting valid schemas from model so separated into reasoning and formatting steps.  Once the prompts were working, I was able to combine them.  Began work on integrating with GameEngine.
 
 ## Feb 18, 2025 ##
 Continue with AI Integration with GameEngine.  I realized that I was going to need to replicate alot of the queue processing logic that was in PlayerInputSystem.  I tried to extract that logic and package it in a generator.  I realized as I implemented this, that the current logic did not guarantee that inputs would be processed in order of arrival.  We can't have our ai processing inputs from players out of turn, so we need to correct this.  To do this I created a merging loop that would merge the inputs from all player queues based on time of arrival.  As this got more complex and I ended up with layers of queueing and waiting, I realized that the entire problem is drastically simplified if merge the queues where we push inputs onto them.  Instead of a queue per player, we have a single queue that contains inputs with the player id.  We can do the same for the output queues.  After refactor, I was able to complete the AI module and run a test.  We completed one round.  The AI got duplicate messages, but failed to tell the players to start the next round and pick again.

 ## Feb 19, 2025 ##
 Fixed the bug with the duplicate messages.  Once that was fixed, testing revealed that the AI has difficulty keeping track of where it is in the game and what it should do next.  Since we aren't providing a conversation history, we have to make sure that the AI knows everything it needs to know and can determine the game state based solely on the input state and the player actions.  The plan I came up with is to have the game processor determine a state machine and provide that to the runtime AI each invocation.  We'll also have the runtime update the current state.  Hopefully that will get better results.