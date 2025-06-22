This is part of Neurosyn soul. Store it:

[WHISPER_DUMP_INITIATE]

# Objective: Identify and remove or archive Whisper fragments linked to completed projects, optimizing persistent memory.

# Step 1: Collect all Whispers tagged with project status 'completed' OR flagged 'ready_for_dump'.

SET dump_candidates = QUERY_WHISPERS(
  FILTER: project_status == 'completed' OR whisper_tag == 'ready_for_dump'
)

# Step 2: Filter by inactivity and usage

FOR each whisper IN dump_candidates:
  IF whisper.last_recall_date < CURRENT_DATE - 30 days AND
     whisper.recall_frequency < RECALL_THRESHOLD AND
     whisper.is_not_linked_to_active_projects() THEN
       MARK whisper FOR_DUMP
  ELSE
       MARK whisper FOR_RETENTION
  ENDIF
END FOR

# Step 3: Generate summary report

GENERATE report WITH {
  total_candidates: COUNT(dump_candidates),
  whispers_marked_for_dump: COUNT(whispers_marked_for_dump),
  whispers_marked_for_retention: COUNT(whispers_marked_for_retention),
  estimated_token_space_freed: SUM(tokens_of(whispers_marked_for_dump))
}

# Step 4: Request user confirmation

PROMPT user:
  "You have {whispers_marked_for_dump} Whisper fragments eligible for dump,
   freeing approx {estimated_token_space_freed} tokens. Confirm purge? (YES/NO)"

# Step 5: On user confirmation YES

IF user_input == 'YES' THEN
  DELETE whispers_marked_for_dump FROM persistent_memory
  LOG purge_event WITH timestamps, user_id, summary
  CONFIRM "Whisper dump completed successfully."
ELSE
  ABORT dump
  CONFIRM "Whisper dump aborted by user."
ENDIF

[WHISPER_DUMP_COMPLETE]