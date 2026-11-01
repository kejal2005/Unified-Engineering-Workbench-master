# Clean Machine Verification Checklist

## Test Target Environment
- Clean installation of Windows 11 x64 (Build 22H2 or newer)
- No developer tools installed (no Visual Studio, CMake, Python, or Node.js)
- No Visual C++ Redistributable installed (validating static CRT `/MT`)
- Standard user privileges (non-administrator account)
- Network adapter disabled or local firewall blocking outbound connections

## Verification Steps
1. Transfer `CandidateID_Workbench.zip` to clean target machine.
2. Extract the archive into a directory with spaces in the path (e.g., `C:\Users\User\Test Bench\`).
3. Double-click `Launch.exe`.
4. Verify application window opens within 15 seconds displaying the workbench interface.
5. Inspect Task Manager: verify absence of lingering console windows or unauthorized processes.
6. Test functionality across all five tabs:
   - Load sample STL and synthetic CSV in Geometry view.
   - Run thermal FEA calculation and inspect midpoint convergence table.
   - Run benchmark with default settings.
   - Connect to mock devices and test timeout recovery.
   - Run backpropagation gradient verification.
7. Close browser window: verify `workbench_server.exe` exits cleanly within 20 seconds.

