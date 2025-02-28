import pandas as pd
import random
import pm4py
from datetime import datetime, timedelta

# Simuliere Event Log für einen Kreditprozess
def generate_event_log(n_cases=1000):
    activities = ["Antrag eingereicht", "Bonitätsprüfung", "Antrag genehmigt", "Vertragsunterzeichnung", "Auszahlung", "Antrag abgelehnt"]
    event_log = []
    
    for case_id in range(1, n_cases+1):
        start_time = datetime(2024, 1, 1) + timedelta(days=random.randint(0, 365))
        approved = random.choice([True, False])
        
        timestamps = [start_time + timedelta(hours=random.randint(1, 48)) for _ in range(5)]
        timestamps.sort()
        
        # Event-Reihenfolge simulieren
        event_log.append([case_id, "Antrag eingereicht", timestamps[0]])
        event_log.append([case_id, "Bonitätsprüfung", timestamps[1]])
        
        if approved:
            event_log.append([case_id, "Antrag genehmigt", timestamps[2]])
            event_log.append([case_id, "Vertragsunterzeichnung", timestamps[3]])
            event_log.append([case_id, "Auszahlung", timestamps[4]])
        else:
            event_log.append([case_id, "Antrag abgelehnt", timestamps[2]])
    
    return pd.DataFrame(event_log, columns=["case_id", "activity", "timestamp"])

# Erstelle und speichere Event Log
event_log_df = generate_event_log()
event_log_df.to_csv("kreditprozess_event_log.csv", index=False)

# Lade Event Log für Process Mining
log = pm4py.format_dataframe(event_log_df, case_id="case_id", activity_key="activity", timestamp_key="timestamp")
log = pm4py.convert_to_event_log(log)

# Prozessmodell Discovery
net, initial_marking, final_marking = pm4py.discover_petri_net_inductive(log)
pm4py.view_petri_net(net, initial_marking, final_marking)
