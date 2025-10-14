 public async Task<IActionResult> GetUserActivityDataWithDynamicSearch([FromBody] GetLoggingDataWithDynamicSearchRequest request)
 {           
     var GetUserActivityDataWithresult = await _adminServices.GetActivityTrackerLogsAsync(request.pageNumber, request.pageSize, request.keyword!);
     if (GetUserActivityDataWithresult == null || GetUserActivityDataWithresult.data == null || GetUserActivityDataWithresult.data.Count == 0)
     {
         return NoContent();
     }
     else
     {
         GetUserActivityDataWithresult.pageSize = request.pageSize;
         GetUserActivityDataWithresult.pageNumber = request.pageNumber;
         return Ok(GetUserActivityDataWithresult);
     }
 }
  public class GetLoggingDataWithDynamicSearchRequest
 {
     [Range(0, 10000000)]
     public int pageNumber { get; set; } = 1;

     [Range(0, 10000)]
     public int pageSize { get; set; } = 20;

     public string? keyword { get; set; } = "";
 }
 public class GetActivityTrackerLogsRepositoryLayerResponse : PaginationRequest
{
    public List<GetActivityTrackerLogsStoredProcedureResponse>? data { get; set; }
}
public class GetActivityTrackerLogsStoredProcedureResponse
{
    public string? firstName { get; set; }
    public string? lastName { get; set; }
    public Guid? sessionID { get; set; }
    public int? employeeID { get; set; }
    public string? screenName { get; set; }
    public string? functionalityID { get; set; }
    public string? functionalityName { get; set; }
    public string? userAction { get; set; }
    public string? affiliateID { get; set; } = string.Empty;
    public string? affiliateName { get; set; } = string.Empty;
    public DateTime? createdOn { get; set; }
    public long? createdOnEpoch { get; set; } = null;
    public string? userAgent { get; set; }
    public string? browserName { get; set; }
    public string? browserVersion { get; set; }
    public string? browserLanguage { get; set; }
    public string? clientIP { get; set; }
    public string? serverIP { get; set; }

}
Task<GetActivityTrackerLogsRepositoryLayerResponse> GetActivityTrackerLogsAsync(int pageNumber, int pageSize, string keyword);
