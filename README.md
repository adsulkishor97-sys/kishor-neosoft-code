public class GroupedData
{
    public string? affiliateId { get; set; }
    public string? name { get; set; }
    public decimal overall { get; set; }
    public decimal critical { get; set; }
    public int overallState { get; set; } = 0;
    public int criticalState { get; set; } = 0;
    public long? overallTarget { get; set; } = null;
    public string? plantId { get; set; }
    public string? plantName { get; set; }

}
