Create a platform-specific start script for the following 2-tier application:

## Application Structure

```
/code
  /backend          → ASP.NET Core Web API (.NET 10.0)
  /frontend         → React + Vite SPA
```

## Requirements

Create **two** scripts in the repository root:

- `start.bat`  → for Windows
- `start.sh`   → for macOS/Linux

## Script Behavior

Both scripts must:
1. Open the **backend** (`/code/backend`) in a **separate terminal window** and run `dotnet run`
2. Open the **frontend** (`/code/frontend`) in a **separate terminal window** and run `npm run dev`
3. Print a short status message before launching each process indicating what is being started

## Platform-specific details

**start.bat (Windows)**
- Use `start` command to open new `cmd` windows for each process
- Each window should have a descriptive title ("Backend" / "Frontend")

**start.sh (macOS/Linux)**
- Detect the available terminal emulator and use it to open new windows
  (prefer: `gnome-terminal`, `xterm` as fallback)
- Make the script executable (`chmod +x` instruction in a comment at the top)

## Additional Requirements
- Both scripts must be placed in the **repository root** (not inside `/code`)
- No Docker, no docker-compose
- No dependency checks or installation steps — assume `dotnet` and `npm` are available