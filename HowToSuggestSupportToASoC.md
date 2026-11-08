## Request/Suggestion

Welcome to the SoC suggestion protocol! If you would like to suggest a SoC to be added to perfd-opt, please follow the protocol below:

### How to suggest SOCs to add to the compatibility list

Submit an issue with these questions answered:

1. Name of your SOC (e.g., sdm680, sdm730)

2. Name of your SOC platform (can be viewed via: `getprop ro.board.platform`)

3. Does your SOC use Schedtune, Uclamp, or both?

4. List the available frequencies of your SOC, for all its clusters if possible.

5. Does your SOC use EAS? (If the answer is not "yes", your issue will be automatically ignored)

### After answering the questions, please send the following files from your SoC:

1. Send your powerhint.xml file, which contains the camera perflocks, etc. It's located in: /vendor/etc

2. Send your boost framework (perfboostconfig.xml) and configuration store (perfconfigstore.xml), located in /vendor/etc/perf

3. Send your thermal daemon (thermal-engine.conf and the version specific to your device), located in /vendor/etc.
