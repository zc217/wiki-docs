---
title: page02
description: 
published: true
date: 2026-02-26T09:24:16.667Z
tags: 
editor: markdown
dateCreated: 2026-02-26T09:24:16.667Z
---

# 考勤

## 自定义调休账户明细有效期

- **上下文**：com.wyc.hr.attendance.timeaccount.lieu.LeaveInLieuTimeAccountDetailValidPeriodContext
- **返回值**：com.wyc.frw.common.datatype.LocalInterval
- **描述**：在加班结转是自定义控制调休账户明细有效期
- **Code**：LEAVE_IN_LIEU_TIME_ACCOUNT_VALID_PERIOD_SCRIPT
- **触发时机**：加班审批通过之后，在运行日报过程中进行加班结转时

## 灵活排班组排班约束检查

- **上下文**：com.wyc.hr.attendance.schedule.team.flex.schedule.request.constraint.FlexScheduleRequestConstraintScriptContext
- **返回值**：java.lang.Void
- **描述**：根据灵活排班组约束方案检查排班申请是否违反约束。
- **Code**：FLEX_SCHEDULE_REQUEST_CONSTRAINT
- **触发时机**：1、手动触发排班申请约束检查；
2、提交排班申请时触发自动检查。

## 考勤方案

### 日报

#### 日报处理脚本

- **上下文**：com.wyc.hr.attendance.report.daily.script.daily.EmployeeDailyReportScriptContextImpl
- **返回值**：java.lang.Void
- **描述**：用于处理日报生成前的处理逻辑
- **Code**：DAILY_REPORT_PROCESS_SCRIPT
- **触发时机**：1.日报生成前执行的预处理脚本; 
2.用于处理日报生成后的后处理逻辑; 
3.处理打卡执行的预处理脚本; 
4.处理打卡执行的后处理脚本

- **示例**：日报处理脚本.groovy
```groovy
package HRM.Attendance.DAILY_REPORT_PROCESS_SCRIPT

import com.wyc.hr.attendance.report.daily.DailyReportInput
import com.wyc.hr.attendance.report.daily.EmployeeDailyReport
import com.wyc.hr.attendance.report.daily.script.ClockResultPostProcessScriptContext
import com.wyc.hr.attendance.report.daily.script.ClockResultPreProcessScriptContext
import com.wyc.hr.attendance.report.daily.script.daily.EmployeeDailyReportScriptContextImpl
/**
 * 打卡结果预处理脚本
 */
def clockResultPreProcess() {
    def context = (ClockResultPreProcessScriptContext) ctx
    DailyReportInput input = context.getInput()
    logger.info("员工ID={}, 日期={}", input.employeeId, input.date)
}


/**
 * 打卡结果后处理脚本
 */
def clockResultPostProcess() {
    def context = (ClockResultPostProcessScriptContext) ctx
    def input = context.getInput()
    def results = context.getClockResults()
    logger.info("员工ID={}, 日期={}", input.employeeId, input.date)
    logger.info("打卡结果={}", results)
}


/**
 * 日报预处理脚本
 */
def preProcess() {
    def context = (EmployeeDailyReportScriptContextImpl) ctx
    DailyReportInput input = context.getDailyReportInput()
    logger.info("员工ID={}, 日期={}", input.employeeId, input.date)
    if (input.previousReport == null) {
        logger.info("无前一日报，跳过预处理")
        return
    }
}


/**
 * 日报处理脚本
 */
def main() {
    def context = (EmployeeDailyReportScriptContextImpl) ctx
    EmployeeDailyReport dailyReport = context.getEmployeeDailyReport()
    DailyReportInput input = context.getDailyReportInput()
    def employee = context.getEmployee()
    logger.info("员工={}, 日期={}", employee.name, dailyReport.date)
}

```

### 月报设置

#### 考勤组月报预处理脚本

- **上下文**：com.wyc.hr.attendance.report.month.script.AttendanceGroupMonthReportScriptContextImpl
- **返回值**：java.lang.Void
- **描述**：在整个考勤组开始运行考勤月报前执行该脚本
- **Code**：ATTENDANCE_GROUP_MONTH_REPORT_PRE_SCRIPT
- **触发时机**：考勤组月报生成前执行的预处理脚本

- **示例**：考勤组月报预处理脚本.groovy
```groovy
package demo.script.groovy.report.month

import com.wyc.hr.attendance.report.month.script.AttendanceGroupMonthReportScriptContext

def main() {
    AttendanceGroupMonthReportScriptContext context = ctx as AttendanceGroupMonthReportScriptContext
    Long attendanceGroupId = context.getAttendanceGroupId()
    String period = context.getPeriod()
    logger.info("考勤组ID: {}", attendanceGroupId)
    logger.info("考勤周期: {}", period)
}

```

#### 考勤组月报执行后脚本

- **上下文**：com.wyc.hr.attendance.report.month.script.AttendanceGroupMonthReportScriptContextImpl
- **返回值**：java.lang.Void
- **描述**：在整个考勤组运行考勤月报结束之后执行该脚本
- **Code**：ATTENDANCE_GROUP_MONTH_REPORT_POST_SCRIPT
- **触发时机**：考勤组月报生成后执行的后处理脚本

- **示例**：考勤组月报执行后脚本.groovy
```groovy
package HRM.Attendance.ATTENDANCE_GROUP_MONTH_REPORT_POST_SCRIPT

import com.wyc.hr.attendance.report.month.script.AttendanceGroupMonthReportScriptContext

def main() {
    AttendanceGroupMonthReportScriptContext context = ctx as AttendanceGroupMonthReportScriptContext
    Long attendanceGroupId = context.getAttendanceGroupId()
    String period = context.getPeriod()
    logger.info("考勤组ID: {}", attendanceGroupId)
    logger.info("考勤周期: {}", period)
}

```

### 月报

#### 月报自定义字段转换脚本

- **上下文**：com.wyc.hr.attendance.report.month.setting.MonthReportUDFDisplayScriptContextImpl
- **返回值**：java.util.Map
- **描述**：在员工端查看考勤月报时，格式化考勤月报中的UDF
- **Code**：MONTH_REPORT_UDF_CONVERT_SCRIPT
- **触发时机**：月报生成时，根据UDF显示设置执行自定义字段转换脚本

- **示例**：月报显示转换脚本.groovy
```groovy
package HRM.Attendance.MONTH_REPORT_UDF_CONVERT_SCRIPT

import com.wyc.hr.attendance.report.month.setting.MonthReportUDFDisplayScriptContext

def main() {
    MonthReportUDFDisplayScriptContext context = ((MonthReportUDFDisplayScriptContext) ctx)
    def udf = context.getUdf()
    udf.put("申请请假次数","1次")
    udf.put("总计请假小时","10小时")
    return udf
}
```

#### 月报扩展脚本

- **上下文**：com.wyc.hr.attendance.report.month.script.EmployeeMonthReportPostScriptContextImpl
- **返回值**：java.lang.Void
- **描述**：在计算完员工考勤月报之后执行该脚本
- **Code**：EMPLOYEE_MONTH_REPORT_SCRIPT
- **触发时机**：在计算完员工考勤月报之后执行该脚本

- **示例**：员工月报扩展脚本.groovy
```groovy
package HRM.Attendance.EMPLOYEE_MONTH_REPORT_SCRIPT

import com.wyc.frw.common.collection.NullSafeIterable
import com.wyc.hr.attendance.masterdata.Employee
import com.wyc.hr.attendance.report.daily.EmployeeDailyReport
import com.wyc.hr.attendance.report.month.MonthReportInput
import com.wyc.hr.attendance.report.month.script.EmployeeMonthReportPostScriptContext
import org.joda.time.LocalDate
import org.joda.time.YearMonth

def main() {
    def context = (EmployeeMonthReportPostScriptContext) ctx

    MonthReportInput input = context.getInput()
    def monthReport = context.getMonthReport()
    Employee employee = context.getEmployee()
    List<EmployeeDailyReport> dailyReports = input.getDailyReports()
    logger.info("员工={}, 月份={}", employee.fullName, input.period)
    def dataMap = [:]
    // 示例1：统计实际出勤天数
    dataMap["实际出勤天数"] = buildActualAttendanceDays(dailyReports)
    // 示例2：计算应出勤天数（固定写法示例）
    dataMap["应出勤天数"] = buildShouldAttendanceDays(input.period)

    // 示例3：计算未满勤天数
    dataMap["未满勤天数"] =
            (dataMap["应出勤天数"] ?: 0) - (dataMap["实际出勤天数"] ?: 0)

    // 输出到月报
    dataMap.each { k, v ->
        logger.info("设置月报字段：{} = {}", k, v)
        monthReport.setUdfValue(k, v)
    }
}
/**
 * 统计实际出勤天数
 */
def buildActualAttendanceDays(List<EmployeeDailyReport> dailyReports) {
    def days = 0
    for (EmployeeDailyReport report : new NullSafeIterable<>(dailyReports)) {
        def attendance = report.getUdfValue("实际出勤天数") ?: 0
        if (attendance > 0) {
            days++
        }
    }
    logger.info("统计实际出勤天数={}", days)
    return days
}

/**
 * 计算应出勤天数
 */
def buildShouldAttendanceDays(String period) {
    YearMonth ym = YearMonth.parse(period)
    LocalDate start = ym.toLocalDate(1)
    LocalDate end = start.plusMonths(1).minusDays(1)
    def days = end.dayOfMonth
    logger.info("计算应出勤天数 = {}", days)
    return days
}

```

### 考勤异常通知

#### 异常通知预处理脚本

- **上下文**：com.wyc.hr.attendance.group.notify.ExceptionNotificationPreProcessScriptContextImpl
- **返回值**：com.wyc.hr.attendance.report.month.notify.EmployeePeriodAttendanceExceptions
- **描述**：在发送考勤异常通知前，处理考勤异常通知内容。
- **Code**：EXCEPTION_NOTIFICATION_PRE_PROCESS
- **触发时机**：异常通知发送前，根据考勤组设置执行预处理脚本

- **示例**：异常通知预处理脚本.groovy
```groovy
package HRM.Attendance.EXCEPTION_NOTIFICATION_PRE_PROCESS

import com.wyc.hr.attendance.group.notify.ExceptionNotificationPreProcessScriptContext
import com.wyc.hr.attendance.report.month.notify.EmployeePeriodAttendanceExceptions
import com.wyc.hr.common.EmployeeStatusEnum
import com.wyc.hr.masterdata.client.dto.EmployeeDetailsDTO

def main() {

    EmployeeDetailsDTO employee = (EmployeeDetailsDTO) ctx.getEmployee()
    if (!isOnboard(employee)) {
        logger.debug("员工不在职，不发送考勤异常通知")
        return null
    }
    def scriptCtx = (ExceptionNotificationPreProcessScriptContext) ctx
    EmployeePeriodAttendanceExceptions exceptions = scriptCtx.getExceptions()
    def chinesePeriod = exceptions.getPeriod().replace("-", "年") + "月"
    exceptions.addCustomizedData("chinesePeriod", chinesePeriod)
    filterExceptions(exceptions)
    return exceptions
}

/**
 * 判断是否在职
 */
def isOnboard(EmployeeDetailsDTO employee) {
    return employee?.currentJobInfo?.employeeStatus == EmployeeStatusEnum.ONBOARD
}

/**
 * 只保留指定的考勤异常类型
 */
def filterExceptions(EmployeePeriodAttendanceExceptions periodExceptions) {
    def keepExceptionNames = [
            "迟到",
            "早退",
            "未打卡"
    ]
    periodExceptions.getExceptions().each { daily ->
        daily.getExceptionNameList().removeIf { name ->
            !keepExceptionNames.contains(name)
        }
    }
    periodExceptions.getExceptions().removeIf {
        it.getExceptionNameList().isEmpty()
    }
}

```

## 考勤设置

### 批量加班规则

#### 批量加班规则-兑付方式推导

- **上下文**：com.wyc.hr.attendance.request.ot.batch.script.BatchOvertimeTransferTypeContext
- **返回值**：com.wyc.hr.attendance.overtime.setting.batch.BatchOvertimeTransferType
- **描述**：批量加班设置中，兑付方式选择由脚本推导时，此脚本用于获取兑付方式
- **Code**：GET_BATCH_OVERTIME_TRANSFER_TYPE
- **触发时机**：批量加班生成加班明细时，如果判断当前批量加班的兑付方式是推导，则调用此脚本获取员工的加班兑付方式

- **示例**：批量加班推导兑付方式.groovy
```groovy
import com.wyc.api.client.internal.InternalOpenApiClientStub
import com.wyc.frw.common.exception.BusinessException
import com.wyc.frw.common.utils.CollectionsUtils
import com.wyc.hr.attendance.OvertimeTypeEnum
import com.wyc.open.api.client.request.md.RunReportRequest

def main() {
    Long employeeId = ctx.getEmployeeBasicInfo().id
    String deptNo = ctx.getEmployeeBasicInfo().deptNo
    def strDate = ctx.getStartTime().toString()[0..9]

    def jobInfo = getEmployeeDataByReport(employeeId, strDate)
    def type = jobInfo.get("用工形式")
    //委外员工默认给加班费
    if (type == "委外") {
        return OvertimeTypeEnum.PAYABLE_OT
    }
    if (type == "新业务外包") {
        return OvertimeTypeEnum.REPLACEABLE_OT
    }
    //没有命中任何一条规则，直接报错
    throw new BusinessException("当前员工没有适用的加班兑付规则，请联系考勤管理员。")
}

def getEmployeeDataByReport(def employeeId, def date) {
    def apiClient = (InternalOpenApiClientStub) ctx.getService("OpenApiClient")
    def request = new RunReportRequest()
    request.setReportName("员工主岗任职信息-日报专用")
    def params = [:]
    params["员工"] = employeeId
    params["日期"] = date
    params["limit"] = 1
    params["offset"] = 0
    request.setParameters(params);
    def result = apiClient.execute(request, true);
    if (CollectionsUtils.isNotEmpty(result.body.resultSet)) {
        return result.body.first()
    }
    return null
}


```

### 加班

#### 加班时长计算脚本

- **上下文**：com.wyc.hr.attendance.overtime.setting.ext.OvertimeScriptContextImpl
- **返回值**：java.util.List
- **描述**：用于计算加班的时长
- **Code**：OVERTIME_CALC_SCRIPT
- **触发时机**：1.计算加班申请时长时执行的脚本；
2.计算加班实际时长时执行的脚本

- **示例**：加班时长计算脚本.groovy
```groovy
package HRM.Attendance.OVERTIME_CALC_SCRIPT

import com.wyc.hr.attendance.client.dto.OvertimeTypeEnum
import com.wyc.hr.attendance.overtime.setting.ext.OvertimeScriptContext
import com.wyc.hr.attendance.request.calculate.OTLineCalculateResult
import com.wyc.hr.common.DurationUtils
import com.wyc.hr.common.IntervalList
import org.joda.time.Interval
import org.joda.time.LocalDate
import org.joda.time.LocalTime

/**
 * calcPlans() 用于提交加班申请时计算加班时长，使用ctx.getPlanCalculateParam()作为计算输入；加班存在多天的情况
 * calcActual() 用于在考勤日报运行时，计算最终的加班时长，使用ctx.getActualCalculateParam()作为计算输入；
 */
/**
 * @return List<OTLineCalculateResult> 如果是 null，表示根据系统的计算规则计算。
 */
def calcPlans() {
    def context = (OvertimeScriptContext) ctx
    def param = context.getPlanCalculateParam()
    def date = param.getStartTime().toLocalDate()
    Interval workInterval = param.getRequestInterval()
    Interval restInterval = new Interval(date.toDateTime(new LocalTime(12, 0, 0)),
            date.toDateTime(new LocalTime(13, 0, 0)))
    IntervalList requestIntervals = new IntervalList(workInterval).subtract(new IntervalList(restInterval))
    def otHours = DurationUtils.toHours(requestIntervals.toDuration())
    
    // 根据加班时长进行分段计算
    if (otHours >= 8) {
        otHours = 8
    } else if (otHours >= 4) {
        otHours = 4
    } else {
        otHours = 0
    }

    def result = new OTLineCalculateResult()
    result.setDate(date)
    result.setPlannedHours(otHours)
    if (isHoliday(date)) {
        result.setTypes([OvertimeTypeEnum.PAYABLE_OT])
    } else {
        result.setTypes([OvertimeTypeEnum.REPLACEABLE_OT])
    }
    return Arrays.asList(result)
}

def calcActual() {
    def param = ctx.getActualCalculateParam()
    def date = param.getStartTime().toLocalDate()

    def clockIntervals = param.getClockIntervals()
    def requestInterval = param.getRequestInterval()
    Interval restInterval = new Interval(date.toDateTime(new LocalTime(12, 0, 0)), date.toDateTime(new LocalTime(13, 0, 0)))
    if (clockIntervals != null && requestInterval != null) {
        IntervalList requestIntervalList = new IntervalList(param.getRequestInterval()).subtract(new IntervalList(restInterval))
        IntervalList clockIntervalList = new IntervalList(param.getClockIntervals()).subtract(new IntervalList(restInterval))
        IntervalList actualIntervalList = requestIntervalList.overlap(clockIntervalList)
        def hours = DurationUtils.toHours(actualIntervalList.toDuration())
        if (hours >= 8) {
            return new BigDecimal(8)
        }
        if (hours >= 4) {
            return new BigDecimal(4)
        }
        return BigDecimal.ZERO
    }
    return BigDecimal.ZERO
}

def isHoliday(LocalDate date) {
    //假设周六周日为假日
    return date.getDayOfWeek() == 6 || date.getDayOfWeek() == 7
}

```

### 假期账户

#### 离职员工时间账户计算脚本

- **上下文**：com.wyc.hr.attendance.timeaccount.calc.ScriptCalculateContext
- **返回值**：java.lang.Object
- **描述**：用于计算离职员工的时间账户余额
- **Code**：TIME_ACCOUNT_QUIT_CALCULATE
- **触发时机**：员工离职时执行

- **示例**：离职折算脚本.groovy
```groovy
package HRM.Attendance.TIME_ACCOUNT_QUIT_CALCULATE

import com.wyc.hr.attendance.masterdata.Employee
import com.wyc.hr.attendance.schedule.employee.setting.EmployeeScheduleSetting
import com.wyc.hr.attendance.script.AttendanceScriptContext
import com.wyc.hr.attendance.timeaccount.TimeAccount

def main() {
    def context = ctx as AttendanceScriptContext
    def timeAccount =  (TimeAccount) context.getVariable("timeAccount")
    def employee =  (Employee) context.getVariable("employee")
    def employeeScheduleSetting =  (EmployeeScheduleSetting) context.getVariable("employeeScheduleSetting")
    List<TimeAccount> allTimeAccounts = context.getVariable("allTimeAccounts") as List<TimeAccount>

    calcTimeAccount(timeAccount)
    return timeAccount;
}

void calcTimeAccount(TimeAccount timeAccount) {
    timeAccount.setAllotted(1.0)
}

```

#### 时间账户分配自定义额度脚本

- **上下文**：com.wyc.hr.attendance.script.AttendanceScriptContextImpl
- **返回值**：com.wyc.hr.attendance.timeaccount.adjust.TimeAccountScriptWrapper
- **描述**：用于分配员工时间账户，自定义额度脚本
- **Code**：TIME_ACCOUNT_ALLOCATION_SCRIPT
- **触发时机**：时间账户分配时执行的脚本

- **示例**：自定义额度脚本.groovy
```groovy
package HRM.Attendance.TIME_ACCOUNT_ALLOCATION_SCRIPT

import com.wyc.hr.attendance.masterdata.Employee
import com.wyc.hr.attendance.schedule.employee.setting.EmployeeScheduleSetting
import com.wyc.hr.attendance.script.AttendanceScriptContext
import com.wyc.hr.attendance.timeaccount.TimeAccount

/**
 * 时间账户分配示例脚本
 */
def main() {

    AttendanceScriptContext context = ctx as AttendanceScriptContext

    TimeAccount timeAccount = context.getVariable("timeAccount") as TimeAccount
    Employee employee = context.getVariable("employee") as Employee
    EmployeeScheduleSetting employeeScheduleSetting = context.getVariable("employeeScheduleSetting") as EmployeeScheduleSetting
    List<TimeAccount> allTimeAccounts = context.getVariable("allTimeAccounts") as List<TimeAccount>

    calcTimeAccount(timeAccount)
    return timeAccount
}


void calcTimeAccount(TimeAccount timeAccount) {
    timeAccount.setAllotted(1.0)
}

```

### 考勤类型设置

#### 考勤类型可见性判断脚本

- **上下文**：com.wyc.hr.attendance.type.AttendanceTypeScriptContextImpl
- **返回值**：java.lang.Boolean
- **描述**：用于判断考勤类型是否对员工可见的脚本逻辑
- **Code**：EMPLOYEE_ATTENDANCE_TYPE_VISIBLE_SCRIPT
- **触发时机**：判断考勤类型是否对员工可见时执行的脚本

- **示例**：考勤类型可见性脚本.groovy
```groovy
package HRM.Attendance.EMPLOYEE_ATTENDANCE_TYPE_VISIBLE_SCRIPT

import com.wyc.hr.attendance.setting.attendance.script.EmployeeAttendanceTypeScriptContext
import com.wyc.hr.masterdata.client.dto.EmployeeDetailsDTO
import com.wyc.hr.common.GenderEnum


def main() {
    EmployeeAttendanceTypeScriptContext scriptCtx = (EmployeeAttendanceTypeScriptContext) ctx
    EmployeeDetailsDTO employee = (EmployeeDetailsDTO) scriptCtx.getEmployee()
    def attendanceType = scriptCtx.getAttendanceType()
    logger.debug("当前考勤类型：${attendanceType?.code}")
    def typeNameList = []
    typeNameList.add("事假")

    // 示例：女性员工可以使用“产检假”
    if (employee.gender == GenderEnum.FEMALE) {
        typeNameList.add("产检假")
    }

    // 示例：根据考勤类型
    if ("LEAVE" == attendanceType?.code) {
        typeNameList.add("年假")
        typeNameList.add("病假")
    }
    return typeNameList
}

```

### 打卡

#### 获取GPS打卡设置脚本

- **上下文**：com.wyc.hr.attendance.clock.setting.ext.ClockSettingContextImpl
- **返回值**：java.util.List
- **描述**：用于获取员工的GPS打卡设置
- **Code**：GET_GPS_CLOCK_SETTINGS
- **触发时机**：获取员工GPS打卡设置时执行的脚本

- **示例**：获取GPS打卡设置脚本.groovy
```groovy
package HRM.Attendance.GET_GPS_CLOCK_SETTINGS

import com.wyc.hr.attendance.clock.setting.ext.ClockSettingContext
import com.wyc.hr.attendance.schedule.employee.setting.EmployeeScheduleSetting

def main() {
    ClockSettingContext context = ctx as ClockSettingContext
    EmployeeScheduleSetting ess = context.getEmployeeScheduleSetting()

    logger.info("获取GPS打卡设置脚本开始执行")
    logger.info("员工ID: {}", ess.getEmployeeId())
    logger.info("考勤组ID: {}", ess.getAttendanceGroupId())

    List<Long> gpsClockSettings = new ArrayList<>()

    if (ess.getAttendanceGroupId() != null) {
        gpsClockSettings.add(1L)
        gpsClockSettings.add(2L)
        gpsClockSettings.add(3L)
    }
    logger.info("GPS打卡设置: {}", gpsClockSettings)
    logger.info("获取GPS打卡设置脚本执行完成")
    return gpsClockSettings
}

```

### 打卡记录集成

#### 外部打卡记录过滤脚本

- **上下文**：com.wyc.hr.attendance.clock.filter.ExternalTimingRecordFilterScriptContextImpl
- **返回值**：java.util.List
- **描述**：用于过滤外部系统导入的打卡记录
- **Code**：EXTERNAL_TIMING_RECORD_FILTER
- **触发时机**：外部系统导入打卡记录时执行的过滤脚本

- **示例**：外部打卡记录过滤脚本.groovy
```groovy
package HRM.Attendance.EXTERNAL_TIMING_RECORD_FILTER

import com.wyc.hr.attendance.clock.filter.ExternalTimingRecordFilterScriptContext
/**
 * 该脚本演示通过第三方集成导入打卡数据的场景下，根据打卡记录的location过滤掉非工作区打卡点数据。
 * 例如，排除掉「园区入口、园区出口」打卡点的打卡数据。
 */
def main() {
    def excludeLocations = ["园区入口", "园区出口"];
    def timingRecords = ((ExternalTimingRecordFilterScriptContext)ctx).getTimingRecords();
    return timingRecords.findAll {!excludeLocations.contains(it.location)}
}


```

## 考勤档案

### 自动化规则

#### 待入职员工考勤档案推导脚本

- **上下文**：com.wyc.hr.attendance.schedule.employee.sync.OnboardingDeductionContext
- **返回值**：java.lang.Void
- **描述**：用于处理待入职员工以及待入职期间任职信息调整时的考勤档案自动推导
- **Code**：ONBOARDING_SCHEDULE_SETTING_DEDUCTION
- **触发时机**：1. 待入职员工事件（EmployeeOnboardingEvent）；
2. 待入职员工任职信息变更事件（JobInfoRevisedEvent）；

- **示例**：待入职考勤档案推导示例脚本.groovy
```groovy
package demo.script.groovy.setting.attendance

import com.wyc.hr.attendance.client.script.EmployeeScheduleSettingDeductionResult
import com.wyc.hr.attendance.schedule.employee.sync.OnboardingDeductionContext
import org.joda.time.LocalDate

def main() {
    OnboardingDeductionContext context = ctx as OnboardingDeductionContext
    def onboardingEvent = context.getOnboardingEvent()
    def revisedEvent = context.getJobInfoRevisedEvent()

    def deduction = new EmployeeScheduleSettingDeductionResult()
    deduction.startDate = LocalDate.now()
    deduction.attendanceGroupName = "默认考勤组"
    deduction.scheduleTeamName = "常规部门排班组"
    deduction.workingHourSystem = "标准工时制"
    deduction.clockRuleName = "打卡规则"
    deduction.overtimeSettingName = "默认加班规则"
    def userDefinedFields = new HashMap<>()
    userDefinedFields.put("测试1", "1")
    userDefinedFields.put("测试2", "2")
    deduction.setUserDefinedFields(userDefinedFields)
    return deduction
}

```

#### 员工考勤档案推导脚本

- **上下文**：com.wyc.hr.attendance.schedule.employee.sync.EmployeeScheduleSettingDeductionContextImpl
- **返回值**：com.wyc.hr.attendance.client.script.EmployeeScheduleSettingDeductionResult
- **描述**：用于处理员工入职以及任职信息调整时的考勤档案自动推导
- **Code**：SCHEDULE_SETTING_DEDUCTION
- **触发时机**：1. 员工入职事件（EmployeeOnBoardEvent）；
2. 员工任职信息生效事件（JobInfoTakeEffectEvent）；

- **示例**：员工考勤档案推导脚本.groovy
```groovy
package HRM.Attendance.SCHEDULE_SETTING_DEDUCTION

import com.wyc.hr.attendance.client.script.EmployeeScheduleSettingDeductionResult
import org.joda.time.LocalDate
import com.wyc.hr.attendance.masterdata.Employee;

def main() {
    def context = ctx as EmployeeScheduleSettingDeductionContext
    Employee employee = context.getEmployee()
    def onBoardEvent = context.getEmployeeOnboardEvent()
    def jobInfoTakeEffectEvent = context.getJobInfoTakeEffectEvent()

    def deduction = new EmployeeScheduleSettingDeductionResult()
    deduction.startDate = new LocalDate(2026, 2, 1)
    deduction.attendanceGroupName = "B1"
    deduction.scheduleTeamName = "常规部门排班组"
    deduction.workingHourSystem = "标准工时制"	//"标准工时制"，"综合工时制"，"不定时工时制"
    deduction.clockRuleName = "打卡规则"
    deduction.overtimeSettingName = "默认加班规则"
    def userDefinedFields = new HashMap<>()
    userDefinedFields.put("测试1","111")
    userDefinedFields.put("测试2","222")
    deduction.setUserDefinedFields(userDefinedFields)
    //deduction.terminateDate = employee.currentJobInfo.startDate.minusDays(1)
    return deduction;
}

```

#### 员工同步过滤脚本

- **上下文**：com.wyc.hr.attendance.script.AttendanceScriptContextImpl
- **返回值**：java.lang.Boolean
- **描述**：员工同步时执行的过滤脚本，用于判断是否需要推导该员工的考勤档案
- **Code**：EMPLOYEE_SYNC_FILTER_SCRIPT
- **触发时机**：1. 员工待入职；
2. 员工入职；

- **示例**：员工同步过滤脚本.groovy
```groovy
package HRM.Attendance.EMPLOYEE_SYNC_FILTER_SCRIPT

import com.wyc.hr.attendance.masterdata.Employee
import com.wyc.hr.attendance.script.AttendanceScriptContext
import com.wyc.hr.common.EmployeeStatusEnum

/**
 * 员工同步过滤脚本
 */
def main() {
    def context = ctx as AttendanceScriptContext
    Employee employee = context.getEmployee()
    if (isEmployeeResigned(employee)) {
        logger.info("员工已离职，不需要同步")
        return false
    }
}

static boolean isEmployeeResigned(Employee employee) {
    return employee.getStatus() == EmployeeStatusEnum.QUIT
}


```

## 出勤

### 员工可用假勤申请

- **上下文**：com.wyc.hr.attendance.setting.attendance.script.EmployeeAttendanceTypeScriptContext
- **返回值**：java.lang.Void
- **描述**：根据员工信息判定可用的请假/其他出勤/请假申请
- **Code**：EMPLOYEE_ENABLED_ATTENDANCE_TYPES
- **触发时机**：员工申请请假/其他出勤/加班

- **示例**：根据员工类型返回排班调整可回溯天数.groovy
```groovy
package HRM.Attendance.EMPLOYEE_ENABLED_ATTENDANCE_TYPES

import com.wyc.hr.attendance.setting.schedule.script.ScheduleAdjustRetrospectiveScriptContext
import com.wyc.hr.masterdata.client.si.VJobInfoBasicDTO

/**
 * 演示如何通过员工类型返回排班调整可回溯天数
 */
def main() {
    def context = ctx as ScheduleAdjustRetrospectiveScriptContext
    for (int i = 0 ; i < context.getEmployeeCount() ; i++) {
        def employee = context.getEmployee(i)
        context.setEmployeeRetrospectiveDays(employee.getId(), getDaysByJobInfo(context.getJobInfoByEmployeeId(employee.getId())))
    }
}

def getDaysByJobInfo(VJobInfoBasicDTO jobInfo) {
    return null != jobInfo.getType() && jobInfo.getType().getCode() == "REGULAR" ?
            7 :
            0
}

```

## 考勤加班

### 自定义调休账户明细有效期

#### 自定义调休账户明细有效期

- **上下文**：com.wyc.hr.attendance.timeaccount.lieu.LeaveInLieuTimeAccountDetailValidPeriodContext
- **返回值**：com.wyc.frw.common.datatype.LocalInterval
- **描述**：在加班结转是自定义控制调休账户明细有效期
- **Code**：LEAVE_IN_LIEU_TIME_ACCOUNT_VALID_PERIOD
- **触发时机**：加班审批通过之后，在运行日报过程中进行加班结转时

- **示例**：自定义调休账户明细有效期.groovy
```groovy
package HRM.Attendance.LEAVE_IN_LIEU_TIME_ACCOUNT_VALID_PERIOD

import com.wyc.frw.common.datatype.LocalInterval
import com.wyc.hr.attendance.timeaccount.lieu.LeaveInLieuTimeAccountDetailValidPeriodContext

/**
 * 演示如何通过员工类型返回调休账户明细有效期
 */
def main() {
    logger.debug("演示如何通过员工类型返回调休账户明细有效期")
    def context = ctx as LeaveInLieuTimeAccountDetailValidPeriodContext
    def startDate = context.getStartDate()
    def jobInfo = context.getJobInfo()
    if (jobInfo.getType().getCode() == "REGULAR") {
        logger.info("正式员工有效期6个月")
        return new LocalInterval(startDate, startDate.plusMonths(6))
    } else {
        logger.info("非正式员工有效期3个月")
        return new LocalInterval(startDate, startDate.plusMonths(3))
    }
}
```

## 考勤排班

### 灵活排班组排班约束检查

#### 灵活排班组排班约束检查

- **上下文**：com.wyc.hr.attendance.schedule.team.flex.schedule.request.constraint.FlexScheduleRequestConstraintScriptContext
- **返回值**：java.lang.Void
- **描述**：根据灵活排班组约束方案检查排班申请是否违反约束。
- **Code**：FLEX_SCHEDULE_CONSTRAINT_RULE
- **触发时机**：1、手动触发排班申请约束检查；
2、提交排班申请时触发自动检查。

- **示例**：灵活排班组排班约束_按人.groovy
```groovy
package HRM.Attendance.FLEX_SCHEDULE_CONSTRAINT_RULE

import com.wyc.frw.common.datatype.LocalInterval
import com.wyc.hr.attendance.schedule.team.flex.schedule.daily.MemberDailySchedule
import com.wyc.hr.attendance.schedule.team.flex.schedule.request.MemberPeriodScheduleRequest
import com.wyc.hr.attendance.schedule.team.flex.schedule.request.constraint.FlexScheduleRequestConstraintScriptContext
import com.wyc.hr.common.collection.NullSafeStream
import org.joda.time.LocalDate

/**
 * 演示如何约束员工连续工作不超过6天，每天工作不超过12小时。
 * 重点：
 * 1、context.getTeamPeriodScheduleRequest().getMemberRequests().each，遍历排班组中的每个成员
 * 2、member.getInterval().each，遍历每一天
 * 3、context.getWorkShift(member.getWorkShiftId(it)) 获取班次
 * 4、context.getPreviousPeriodEmployeeSchedules().get(it.getEmployeeId()) 获取员工上个周期的排班
 * 5、context.addMemberViolation(employeeId, message) 记录违反的约束
 */

class Config {
    static final int DAYS = 6
    static final BigDecimal HOURS = new BigDecimal(12)
}

def main() {
    should_not_work_more_than_X_hours(ctx)
    should_not_continuous_work_more_than_X_days(ctx)
}

def should_not_work_more_than_X_hours(FlexScheduleRequestConstraintScriptContext context) {
    NullSafeStream.of(context.getTeamPeriodScheduleRequest().getMemberRequests()).each { member ->
        member.getInterval().each {
            def workShift = context.getWorkShift(member.getWorkShiftId(it))
            def planHours = null != workShift ? workShift.getPlanWorkHours(null) : BigDecimal.ZERO
            if (planHours > Config.HOURS) {
                context.addMemberViolation(member.getEmployeeId(),
                        "日期：${it}；班次：${workShift.getName()}；班次时长：${workShift.getPlanWorkHours(null)}，超过${Config.HOURS}小时")
            }
        }
    }
}

def should_not_continuous_work_more_than_X_days(FlexScheduleRequestConstraintScriptContext context) {
    def request = context.getTeamPeriodScheduleRequest()
    NullSafeStream.of(request.getMemberRequests()).each {
        employee_should_not_continuous_work_more_than_X_days(
                it,
                context.getPreviousPeriodEmployeeSchedules().get(it.getEmployeeId()),
                context)
    }
}

def employee_should_not_continuous_work_more_than_X_days(MemberPeriodScheduleRequest request,
                                                         List<MemberDailySchedule> prevSchedules,
                                                         FlexScheduleRequestConstraintScriptContext context) {
    Map<LocalDate, Long> workShiftIdMap = [:]
    NullSafeStream.of(prevSchedules).each { workShiftIdMap.put(it.getDate(), it.getWorkShiftId()) }
    request.interval.each { workShiftIdMap.put(it, request.getWorkShiftId(it)) }

    def dateIdx = null
    def days = 0
    new LocalInterval(request.getStartDate().minusDays(Config.DAYS - 1), request.getEndDate()).each {
        def workShift = context.getWorkShift(workShiftIdMap.get(it))
        if (null == workShift || workShift.isOffDay()) {
            if (days > Config.DAYS) {
                context.addMemberViolation(request.getEmployeeId(),
                        "日期：${dateIdx}至${it.minusDays(1)}，连续工作${days}天，超过${Config.DAYS}天")
            }
            days = 0;
            dateIdx = null
        } else {
            if (null == dateIdx) dateIdx = it
            days++
        }
    }
    if (days > Config.DAYS) {
        context.addMemberViolation(request.getEmployeeId(),
                "日期：${dateIdx}至${request.getEndDate()}，连续工作${days}天，超过${Config.DAYS}天")
    }
}
```

- **示例**：灵活排班组排班约束_按岗.groovy
```groovy
package HRM.Attendance.FLEX_SCHEDULE_CONSTRAINT_RULE


import com.wyc.hr.attendance.schedule.team.flex.schedule.request.constraint.FlexScheduleRequestConstraintScriptContext

/**
 * 演示如何约束排班岗位（服务岗）UDF控制人数限制。
 * 重点：
 * 1、context.getPositionPerspectiveRequests() 获取排班岗位（服务岗）维度的排班申请，遍历每个排班岗位（服务岗）维度的排班申请
 * 2、request.getDailySchedules() 遍历每一天排班的人数
 * 3、context.getSchedulePosition(request.getSchedulePositionId())获取排班岗位（服务岗）
 * 4、context.addPositionViolation(positionId, message) 添加服务岗约束 violations
 */

def main() {
    should_have_enough_members(ctx)
}

def should_have_enough_members(FlexScheduleRequestConstraintScriptContext context) {
    context.getPositionPerspectiveRequest().each {request ->
        def schedulePosition = context.getSchedulePosition(request.getSchedulePositionId())
        def min = 5
        def max = 15
        request.getDailySchedules().forEach {
            if (it.getEmployeeCount() < min || it.getEmployeeCount() > max) {
                context.addPositionViolation(schedulePosition.getId(),
                        "服务岗：${schedulePosition.getName()}；日期：${it.getDate()}，安排了${it.getEmployeeCount()}个员工。不满足${min}~${max}人数限制")
            }
        }
    }
}
```

### 灵活排班约束方案选择

#### 灵活排班约束方案选择

- **上下文**：com.wyc.hr.attendance.setting.schedule.script.ScheduleConstraintSchemeScriptContext
- **返回值**：java.lang.Void
- **描述**：用于确定排班组适用的排班约束方案。
- **Code**：SCHEDULE_CONSTRAINT_SCHEME
- **触发时机**：灵活排班组排班申请提交；手动触发灵活排班组排班约束检查。

- **示例**：灵活排班约束方案选择.groovy
```groovy
package HRM.Attendance.SCHEDULE_CONSTRAINT_SCHEME

import com.wyc.hr.attendance.setting.schedule.script.ScheduleConstraintSchemeScriptContext
import com.wyc.hr.attendance.schedule.team.flex.FlexScheduleTeam
/**
 * 演示如何通过排班组信息确定对应的排班约束方案。
 * 1、获取灵活排班组中的「专业」udf
 * 2、根据「专业」返回不同专业对应的约束方案编码
 */
def main() {
    def context = ctx as ScheduleConstraintSchemeScriptContext
    def scheduleTeam = context.getTeam() as FlexScheduleTeam
    def major = scheduleTeam.getUdfValue("专业") as String
    if (null == major) {
        return null
    }
    if ("秩序" == major) {
        return "秩序排班约束方案_编码"
    } else if ("保洁" == major) {
        return "保洁排班约束方案_编码"
    } else {
        return "默认约束方案_编码"
    }
}

```

### 灵活排班组人员

#### 灵活排班组人员自动纳入规则

- **上下文**：com.wyc.hr.attendance.schedule.team.flex.script.FlexScheduleTeamMemberOnEventScriptContext
- **返回值**：java.lang.Void
- **描述**：通过监听相关事件，确定是否将人员加入到特定的灵活排班组
- **Code**：FLEX_SCHEDULE_TEAM_MEMBER_ON_EVENT
- **触发时机**：1, 入职事件; 
2, 员工任职信息（包括主岗和兼岗）调整生效事件; 
3, 员工任职异动（包括主岗和兼岗）更新事件; 
4, 员工离职生效; 
5, 员工新增兼岗生效事件; 
6, 员工兼岗结束生效事件;
7, 员工待入职事件;
8, 员工待入职取消事件;
9, 待入职任职信息修改事件

- **示例**：灵活排班组人员增量处理脚本.groovy
```groovy
package HRM.Attendance.FLEX_SCHEDULE_TEAM_MEMBER_ON_EVENT

import com.wyc.frw.common.exception.BusinessException
import com.wyc.frw.common.log.Logger
import com.wyc.hr.attendance.schedule.team.flex.script.FlexScheduleTeamMemberOnEventScriptContext
import com.wyc.hr.attendance.schedule.team.flex.script.FlexScheduleTeamMemberScriptUtils
import com.wyc.hr.masterdata.client.event.*
import com.wyc.hr.masterdata.org.client.event.JobInfoAdjustTakeEffectiveEvent 
/**
 * 灵活排班组人员自动纳入规则
 * 触发脚本的事件包括：
 * 1, 入职事件;
 * 2, 员工任职信息（包括主岗和兼岗）调整生效事件;
 * 3, 员工任职异动（包括主岗和兼岗）更新事件;
 * 4, 员工离职生效;
 * 5, 员工新增兼岗生效事件;
 * 6, 员工兼岗结束生效事件
 * 7, 员工待入职事件
 * 8, 待入职取消事件
 * 9, 待入职任职信息修改事件
 */

def main() {
    def context = ctx as FlexScheduleTeamMemberOnEventScriptContext
    FlexScheduleTeamMemberEventHandler eventHandler = new FlexScheduleTeamMemberEventHandler(context, logger)
    if (context.getEvent() instanceof EmployeeOnBoardEvent) {
        eventHandler.onEmployeeOnboard(context.getEvent() as EmployeeOnBoardEvent)
    } else if (context.getEvent() instanceof JobInfoAdjustTakeEffectiveEvent) {
        eventHandler.onJobInfoAdjustTakeEffective(context.getEvent() as JobInfoAdjustTakeEffectiveEvent)
    } else if (context.getEvent() instanceof JobInfoChangeRequestUpdatedEvent) {
        eventHandler.onJobInfoChangeRequestUpdated(context.getEvent() as JobInfoChangeRequestUpdatedEvent)
    } else if (context.getEvent() instanceof EmployeeQuitEffectiveEvent) {
        eventHandler.onEmployeeQuitEffective(context.getEvent() as EmployeeQuitEffectiveEvent)
    } else if (context.getEvent() instanceof NewAdditionalJobTakeEffectiveEvent) {
        eventHandler.onNewAdditionalJobTakeEffective(context.getEvent() as NewAdditionalJobTakeEffectiveEvent)
    } else if (context.getEvent() instanceof EndAdditionalJobTakeEffectiveEvent) {
        eventHandler.onEndAdditionalJobTakeEffective(context.getEvent() as EndAdditionalJobTakeEffectiveEvent)
    } else if (context.getEvent() instanceof EmployeeOnboardingEvent) {
        eventHandler.onEmployeeOnboarding(context.getEvent() as EmployeeOnboardingEvent)
    } else if (context.getEvent() instanceof EmployeeCancelOnboardEvent) {
        eventHandler.onEmployeeOnboardingCancel(context.getEvent() as EmployeeCancelOnboardEvent)
    } else if (context.getEvent() instanceof JobInfoRevisedEvent) {
        eventHandler.onEmployeeJobInfoRevisedEvent(context.getEvent() as JobInfoRevisedEvent)
    }
}

class FlexScheduleTeamMemberEventHandler {

    FlexScheduleTeamMemberOnEventScriptContext context
    FlexScheduleTeamMemberScriptUtils utils
    Logger logger

    FlexScheduleTeamMemberEventHandler(FlexScheduleTeamMemberOnEventScriptContext context, Logger logger) {
        this.context = context
        this.utils = context.getUtils()
        this.logger = logger
    }

    def onEmployeeOnboard(EmployeeOnBoardEvent event) {
        logger.info("员工入职事件：" + event);
        throw new BusinessException("for debug purpose")
    }

    def onJobInfoAdjustTakeEffective(JobInfoAdjustTakeEffectiveEvent event) {
        logger.info("员工任职信息调整生效事件：" + event);
        throw new BusinessException("for debug purpose")
    }

    def onJobInfoChangeRequestUpdated(JobInfoChangeRequestUpdatedEvent event) {
        logger.info("员工任职异动更新事件：" + event);
        throw new BusinessException("for debug purpose")
    }

    def onEmployeeQuitEffective(EmployeeQuitEffectiveEvent event) {
        logger.info("员工离职生效事件：" + event);
        throw new BusinessException("for debug purpose")
    }

    def onNewAdditionalJobTakeEffective(NewAdditionalJobTakeEffectiveEvent event) {
        logger.info("员工新增兼岗生效事件：" + event);
        throw new BusinessException("for debug purpose")
    }

    def onEndAdditionalJobTakeEffective(EndAdditionalJobTakeEffectiveEvent event) {
        logger.info("员工兼岗结束生效事件：" + event);
        throw new BusinessException("for debug purpose")
    }

    def onEmployeeOnboarding(EmployeeOnboardingEvent event) {
        logger.info("待入职员工事件：" + event)
        throw new BusinessException("for debug purpose")
    }

    def onEmployeeOnboardingCancel(EmployeeCancelOnboardEvent event) {
        logger.info("待入职取消事件：" + event)
        throw new BusinessException("for debug purpose")
    }
    
    def onEmployeeJobInfoRevisedEvent(JobInfoRevisedEvent event) {
        logger.info("待入职任职信息修改事件：" + event)
        throw new BusinessException("for debug purpose")
    }
}
```

### 排班人员可调整回溯天数

#### 排班人员可调整回溯天数

- **上下文**：com.wyc.hr.attendance.setting.schedule.script.ScheduleAdjustRetrospectiveScriptContext
- **返回值**：java.lang.Void
- **描述**：获取排班人员可调整回溯天数
- **Code**：EMPLOYEE_SCHEDULE_ADJUST_RETROSPECTIVE
- **触发时机**：排班过程中查询排班组中每个成员的调整排班可回溯天数

- **示例**：根据员工类型返回排班调整可回溯天数.groovy
```groovy
import com.wyc.hr.attendance.setting.schedule.script.ScheduleAdjustRetrospectiveScriptContext
import com.wyc.hr.masterdata.client.si.VJobInfoBasicDTO

/**
 * 演示如何通过员工类型返回排班调整可回溯天数
 */
def main() {
    def context = ctx as ScheduleAdjustRetrospectiveScriptContext
    for (int i = 0 ; i < context.getEmployeeCount() ; i++) {
        def employee = context.getEmployee(i)
        context.setEmployeeRetrospectiveDays(employee.getId(), getDaysByJobInfo(context.getJobInfoByEmployeeId(employee.getId())))
    }
}

def getDaysByJobInfo(VJobInfoBasicDTO jobInfo) {
    return null != jobInfo.getType() && jobInfo.getType().getCode() == "REGULAR" ?
            7 :
            0
}

```

## 按岗排班

### 自定义约束指标

#### 自定义约束指标

- **上下文**：com.wyc.hr.attendance.schedule.position.schedule.constraint.ScheduleConstraintScriptContext
- **返回值**：com.wyc.hr.attendance.schedule.position.schedule.constraint.ScheduleConstraint
- **描述**：用于定义自定义约束
- **Code**：SCHEDULE_CONSTRAINT_INDICATOR
- **触发时机**：1、在执行按岗排班时。2、修改按岗排班之后检查排班结果。

- **示例**：按岗排班自定义约束指标脚本示例.groovy
```groovy
package HRM.Attendance.SCHEDULE_CONSTRAINT_INDICATOR


import org.joda.time.LocalDate;

import com.wyc.frw.common.datatype.LocalInterval;
import com.wyc.hr.attendance.schedule.constraint.ScheduleConstraintItem;
import com.wyc.hr.attendance.schedule.position.schedule.rule.EmployeeResource;
import com.wyc.hr.attendance.schedule.position.schedule.rule.PositionShift;
import org.springframework.util.CollectionUtils;
import com.wyc.hr.attendance.schedule.position.schedule.constraint.ScheduleValidatorResult

public class TestConstraint extends AbstractEmployeeScopeConstraint {
	private List<LocalInterval> continuousWorkingDates = new ArrayList<>();
	
	public TestConstraint(ScheduleConstraintItem constraint, EmployeeResource eeResource) {
		super(constraint, eeResource);
		constraint.setLowerLimit(0);
		constraint.setUpperLimit(4);
	}
	
	private BigDecimal maxContinuousWorkingDays(List<LocalInterval> list)
	{
		int max = 0;
		for(LocalInterval interval: list)
		{
			if(interval.days() > max)
				max = interval.days();
		}
		
		return new BigDecimal(max);
	}

	@Override
	public void onEmployeeAssigned(PositionShift positionShift, EmployeeResource employeeResource) {
		List<LocalInterval> list = this.mergeIntervals(this.continuousWorkingDates, positionShift.getDate());
		this.continuousWorkingDates = list;
	}

	public List<LocalInterval> mergeIntervals(List<LocalInterval> intervals, LocalDate date) {
		List<LocalInterval> mergedIntervals = new ArrayList<>();
		if (!CollectionUtils.isEmpty(intervals)) {
			mergedIntervals.addAll(intervals);
		}
		mergedIntervals.add(new LocalInterval(date, date));
		return LocalInterval.concat(mergedIntervals);
	}

	@Override
	public boolean meetConstraintIfAssigned(PositionShift shift, EmployeeResource employeeResource) {
		List<LocalInterval> newList = this.mergeIntervals(continuousWorkingDates, shift.getDate());
		BigDecimal newVal = this.maxContinuousWorkingDays(newList);
		return this.constraint.meet(newVal);
	}

	@Override
	public void onValidate(ScheduleValidatorResult result) {
		BigDecimal currentVal = this.maxContinuousWorkingDays(this.continuousWorkingDates);
		if(!this.constraint.meet(currentVal))
		{
			result.addErrMsg(String.format("员工【%s】连续排班天数为【%s】，超过最大连续排班天数【%s】", super.getEmployeeNameAndCode(),
					currentVal, this.constraint.getUpperLimit()));
		}
	}

}


def main(){
 return new TestConstraint(ctx.getConstraintItem(), ctx.getEmployeeResource());
}

```

## 排班

### 班次管理

#### 无固定时间班次打卡时间脚本

- **上下文**：com.wyc.hr.attendance.workshift.schedule.ext.NotFixTimeScheduleScriptContextImpl
- **返回值**：java.util.List
- **描述**：用于解析无固定时间班次的打卡时间
- **Code**：NOT_FIX_TIME_WORK_SHIFT_CLOCK_TIMING
- **触发时机**：无固定时间班次打卡时执行的脚本

- **示例**：无固定时间班次打卡时间脚本.groovy
```groovy
package HRM.Attendance.NOT_FIX_TIME_WORK_SHIFT_CLOCK_TIMING

import com.wyc.hr.attendance.clock.TimingRecord
import com.wyc.hr.attendance.workshift.NotFixTimeWorkShiftItem
import com.wyc.hr.attendance.workshift.schedule.ext.NotFixTimeScheduleScriptContextImpl
import org.apache.commons.collections.CollectionUtils
/**
 * 无固定时间班次打卡时间脚本
 */
def main() {
    def context = ctx as NotFixTimeScheduleScriptContextImpl

    NotFixTimeWorkShiftItem workShiftItem = context.getWorkShiftItem()
    List<TimingRecord> timingRecords = context.getTimingRecords()

    if (CollectionUtils.isEmpty(timingRecords)) {
        return []
    }

    List<TimingRecord> validRecords = filterInvalid(timingRecords)
    validRecords.sort { a, b -> a.dateTime <=> b.dateTime }
    List<TimingRecord> result = trimRecords(validRecords, 6)
    return result
}

List<TimingRecord> filterInvalid(List<TimingRecord> records) {
    return records.findAll { it != null && it.dateTime != null }
}

List<TimingRecord> trimRecords(List<TimingRecord> records, int maxSize, def log) {
    if (records.size() <= maxSize) {
        return records
    }
    return records.subList(0, maxSize)
}

```

