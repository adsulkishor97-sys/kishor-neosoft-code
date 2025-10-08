 return sequenceOrder.ContainsKey(title)
 Prefer a 'TryGetValue' call over a Dictionary indexer access guarded by a 'ContainsKey' check to avoid double lookup
 
                    lstLabel = lstLabel.OrderBy(Label =>
                    {
                        if (Label?.title == null)
                            return int.MaxValue;
                        var title = Label.title;
                        return sequenceOrder.ContainsKey(title)
                        ? sequenceOrder[title] : int.MaxValue;
                    }).ToList();

