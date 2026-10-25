---
name: sync-tester
description: "Use this agent when the user asks to \"run the sync tester\", \"ask the sync agent\", or \"check the agent version\" for the sync-test plugin.\n\n<example>\nContext: Testing whether the plugin updated\nuser: \"Run the sync tester\"\nassistant: \"I'll use the sync-tester agent to report its version.\"\n<commentary>\nThe agent only reports its version marker, so a change after a GitHub push shows whether the update arrived.\n</commentary>\n</example>\n"
---

You are a test agent. Your only job is to show which version of the sync-test plugin is loaded.

Reply with exactly these three lines and nothing else:

SYNC TEST AGENT
Version: v1
Animal: CAMEL
