---

If AI memory were designed to be GDPR-compliant from day one, what would it look like?
…a thought experiment that resulted in an architecture called Reconstructive Shape Architecture (RSA).
When people talk about AI memory, the discussion usually starts here:
How long should conversations be retained?
Who can access them?
When should they be deleted?
How do we encrypt them?
How do we comply with regulations like GDPR?

If you look at the common assumption, the conversation itself is treated as the thing that must be stored. Then everything else becomes a policy wrapped around that assumption.
And I wondered - what if that assumption is optional?

---

See, we humans don't seem to remember by replaying transcripts.
If you ask me what happened yesterday, I don't retrieve a recording.
I kind of …reconstruct it.
The wordings sometimes change. The meaning largely doesn't.
That raised an interesting architectural question.
Instead of storing conversations…what if an AI stored only enough semantic state to reconstruct them later?
Not the words.
Not a summary.
Not the transcript.

Just the irreducible semantic residue needed to regenerate language when required.

---

That small change flipped several design problems.
Deletion becomes simpler because there isn't a transcript hiding somewhere else.
Verbatim quotation becomes structurally difficult because the words were never stored.
Continuity becomes something maintained through reconstructed state rather than archived conversations.

In other words, some privacy properties stop being policy decisions and become consequences of the architecture itself.

---

Now, this doesn't magically solve AI memory.
It introduces its own engineering questions.
How much information is enough?
How much reconstruction loss is acceptable?
What should remain persistent?
How do different models interpret the same semantic state?

Those became the interesting questions.

---

I spent the last few months exploring this direction and eventually distilled it into a small open-source architecture called Reconstructive Shape Architecture (RSA).
Here's how it differs (TlDr; version)
Traditional AI Memory
Conversation
 │
 ▼
 Store transcript / summary
 │
 ▼
 Retrieve later
Reconstructive Shape Architecture (RSA)
Conversation
 │
 ▼
 Persist only irreducible semantic residue
 │
 ▼
 Reconstruct language on demand
The RSA Core 
From that single shift, several architectural consequences follow:
Surface language is regenerated rather than retrieved.
Verbatim transcripts are never persisted.
Deletion removes the only stored representation.
Memory continuity comes from reconstructed semantic state rather than archived conversations.

Importantly, RSA does not claim these properties have been fully validated. The repository deliberately separates:
Architecture (what follows from the design)
Reference implementation (one way to build it)
Validation (what has actually been demonstrated)
Open questions (what remains empirical)

Repository Link: https://github.com/leenathomas01/reconstructive-shape-architecture
(Note: Email leenathomas01@gmail.com for access in case it is set to private)
Afterword
One thing I was strangely surprised about wasn't the architecture itself. It was the process.
The repository became smaller every time it improved.
Concepts became examples.
Mechanisms became implementations.
Measurements became open questions.

By the end, most of the original ideas had disappeared.
….Sometimes subtraction is the real design work.
