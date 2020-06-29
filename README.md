# SimpleCheatManager
A plug-and-play UE4 CheatManager extension that allows for the creation of cheats for debugging/prototyping in a multiplayer game. Currently only compatible with c++ projects.
Documentation is provided in source as well as here.

# 1.) Enabling the plugin
- Clone this repo
- Move SimpleCheatPlugin/ into {PathToYourGame}/{YourGame}/Plugins/
- Open your project enable the plugin in Edit->Plugins. The plugin should appear under "Project->Other".
- You should now be able to create classes extending from SimpleCheat_

# 2.) Setting up for cheats.
- Derive from SimpleCheatManager.
- Attach a SimpleCheatComponent to your currently used PlayerController (designated in the active GameMode for your current map).
- In the details pane of your Playercontroller class (or the constructor of the PlayerController class, if c++) change the CheatClass to be your derived type.
- Verify everything is setup correctly by beginning play and opening the console command ` and submitting SimpleCheatTest. If you are running a networked PIE instance, an Authority and Client string should appear on screen. If you are not running a networked PIE instance, only an Authority string should appear on screen.

# 3.) Running existing cheats.
- As shown above, you can run cheats through the console command `.
- The way cheats are ran depends on how they were implemented in your derived CheatManager class. For example a cheat declared with the following method signature: `vold GiveGold(float Amount)` will be ran by "GiveGold 50" in the console window.

# 4.) Writing new cheats.
- Declare a new cheat with UFUNCTION(Exec) in YourCheatManager.h, which should be derived from SimpleCheatManager. 
- Define the function in YourCheatManager.cpp, doing anything you wish inside the function body. You can get a reference to the player running the cheat with GetOuterAPlayerController(). You can get a reference to the world via GetWorld().
- To run cheats on the server use the provided ServerCheat() method. You must provide it with a string equal to the console command inputted to run the cheat. For example, a cheat defined with GiveMaxHealth(float HealthToGive) would be passed through ServerCheat as "GiveMaxHealth 50". I recommend using the helper macro FUNC_NAME to grab the name of the current function when creating arguments for ServerCheat.

# 5.) Example cheats

1.) Server-Only grant gold.

.h

```
UFUNCTION(Exec)
void GrantGold(float Amount);
```

.cpp
```
void UMySimpleCheatManager::GrantGold(float Amount)
{

  // Send off to server. Server will also run this code, so we only want to run this code if we're on the client. 
  // Not doing this will cause this method to infinitely re-enter itself.
  if(!HasAuthority())
  {
    ServerCheat(FString::Printf(TEXT("%s %f"), *FUNC_NAME, Amount));
    return;
  }
  
  // This is the code that we intend for the server to run. 
  APlayerController* OuterPC = GetOuterAPlayerController();
  OuterPC->GetPawn()->FindComponentByClass<UMyGoldComponent>()->GrantGold(Amount);

}
```

# 6.) Common errors.
If for some reason you followed the above steps but cheats are still not being ran, errors can generally be found in the console under the log category `LogSimpleCheatPlugin`.

1.) Command not recognized: {MyCheat}
- You have a typo in the cheat, or the CheatManager defining the cheat is not currently being instantiated on the PlayerController receiving the console command.

2.) Owner is not a Player Controller. SimpleCheatComponent should only be attached to PlayerControllers.
- The SimpleCheatComponent has found that it is attached to something that is not a PlayerController. Attach it to a PlayerController.

3.) Owner does not have an instantiated CheatManager. Cheats cannot be ran without an instantiated CheatManager.
- Ensure the PlayerController which the SimpleCheatComponent is attached to has a CheatManager specified. Ensure it is being instantiated.

4.) Outer SimpleCheatComponent not found. Add a USimpleCheatComponent to the outer PlayerController.
- Ensure that the PlayerController that the CheatManager is associated with has a SimpleCheatComponent attached to it.

5.) I cannot make a class derived from USimpleCheatManager or USimpleCheatComponent.
- Ensure that the plugin is enabled, see steps above.
