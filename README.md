class SqlKpiData
{
    public string kpiCode { get; set; }
    public int? direction { get; set; }
    public long? overallTargetMax { get; set; }
}
var kpiJsonList = await _benchMarkRepository.GetBusinessKPIDetails();
var sqlKpiDataList = kpiJsonList.Select(x => new SqlKpiData
{
    kpiCode = x.kpiCode,
    direction = x.direction,
    overallTargetMax = x.overallTargetMax
}).ToList();

var kpiinfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);
AddComputedFields(groupData, sqlKpiDataList, request.kpiCode, kpiinfo);

