 return sequenceOrder.ContainsKey(title)
 Prefer a 'TryGetValue' call over a Dictionary indexer access guarded by a 'ContainsKey' check to avoid double lookup

