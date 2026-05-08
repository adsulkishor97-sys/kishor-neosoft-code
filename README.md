public class AssetFailureResponse
{
    public string subCategory { get; set; }
    public string? failure_methodology { get; set; }
    public List<FailureResponse> failures { get; set; }
    public List<KpiResponse> kpis { get; set; }
}

public class FailureResponse
{
    public string month { get; set; }
    public string label { get; set; }
    public int count { get; set; }
}

public class KpiResponse
{
    public string kpi_code { get; set; }
    public string kpi_name { get; set; }
    public int direction { get; set; }
    public List<RollingAverageResponse> rolling_averages { get; set; }
}

public class RollingAverageResponse
{
    public int period_months { get; set; }
    public List<KpiDataResponse> data { get; set; }
}

public class KpiDataResponse
{
    public string month { get; set; }
    public double value { get; set; }
    public long timeEpoch { get; set; }
}
