[TOC]



# ����
## ��������
1. boost�� logģ���Ǿ�̬����ʽ����, ��������ʱ��Ҫ���Ӷ����boost_log_setup boost_log boost_thread boost_system;
2. ����ѡ�Ҫ����`-DBOOST_LOG_DYN_LINK`, ����Ҳ�ᱨ��;
3. ����boost����ļ�, �����ٶȾͻ����, ����ʹ��gcc -H�����鿴�����������е��ļ�, ��֪���ж�����; ����취: ʹ��ͷ�ļ�Ԥ���뼼�����Ժ�, ��ʱû���ҵ����õİ취;

## �����ռ�
Ϊ�˴�����д����, �Լ���������; һ�㶼��������ռ䶨��һЩ����, ����:

```
namespace logging = boost::log;
namespace src = boost::log::sources;
namespace sinks = boost::log::sinks;
namespace keywords = boost::log::keywords;
namespace attrs = boost::log::attributes;
namespace expr = boost::log::expressions;
```


# ʹ��
## ����ʹ��
����־�ȼ�����boost log�Ļ���ʹ��ʵ������:
```C++
namespace logging = boost::log;

void Init()
{
	//logging::core::get() returns a pointer to the core singleton
	logging::core::get()->set_filter(logging::trivial::severity >= logging::trivial::info);
}

void TestBoostLog()
{
	Init();
	BOOST_LOG_TRIVIAL(trace) << "A trace severity message";
	BOOST_LOG_TRIVIAL(debug) << "A debug severity message";
	BOOST_LOG_TRIVIAL(info) << "An informational severity message";
	BOOST_LOG_TRIVIAL(warning) << "A warning severity message";
	BOOST_LOG_TRIVIAL(error) << "An error severity message";
	BOOST_LOG_TRIVIAL(fatal) << "A fatal severity message";
}
```

���:
```
[2017-04-14 14:33:03.427867] [0x00007ffff7fd99c0] [info]    An informational severity message
[2017-04-14 14:33:03.427909] [0x00007ffff7fd99c0] [warning] A warning severity message
[2017-04-14 14:33:03.427918] [0x00007ffff7fd99c0] [error]   An error severity message
[2017-04-14 14:33:03.427925] [0x00007ffff7fd99c0] [fatal]   A fatal severity message
```
�����Ϣ����:
1. ʱ���;
2. ��ǰ�̵߳�ID;
3. ��־�ĵȼ�;
4. Message, ��־����;

**˵��:**
1. Trivial ͷ�ļ�������һ��Ŀ���̨���, ��־��Ĭ�ϵȼ�Ҳ�����ͷ�ļ�, ��Ҳ��boost log��Ĭ�ϵ���־�ȼ�;
```
//! Trivial severity levels
enum severity_level
{
    trace,
    debug,
    info,
    warning,
    error,
    fatal
};
```
2. ȫ����־�ȼ���������ʹ��`logging::core::get()->set_filter`;
3. ע��`set_filter`����, ʹ�õ���[Boost.Phoenix](http://www.boost.org/doc/libs/1_63_0/libs/phoenix/doc/html/index.html)  lambda���ʽ; ���ʽ����߲�������һ���ȴ���У�������, ���ʽ���ұ�����Ҫ��У���ֵ;
4. `logging::core::get()` ������һ��ָ��ָ��core������;

## ��־���
������־ֱ������ڿ���̨, �������־ֱ�����������̨�϶�����ȡ, boost log֧���Զ������, ͨ������Sinksע�ᵽlogging codeʵ��;

ע��: ��������, û�г�ʼ���κ�sink, ��Ȼ��������, ������Ϊlog�����һ��Ĭ�ϵ�sink, ��sink�Թ̶��ĸ�ʽ��ӡ����Ļ��;

### �ļ����

```
void Init()
{
	logging::add_file_log(
		keywords::file_name = "sample_%N.log",
		keywords::rotation_size = 1 * 1024 * 1024,
		keywords::time_based_rotation = sinks::file::rotation_at_time_point(0, 0, 0),
		keywords::format = "[%TimeStamp%]: %Message%"           
		);
	//logging::core::get() returns a pointer to the core singleton
	logging::core::get()->set_filter(logging::trivial::severity >= logging::trivial::info);
}
```
˵��:
* ���ǿ���ʹ��`logging::add_file_log`��ʼ��sink, ����log record ���ļ���;
* �ú�����������һЩ�Զ���ѡ��:
    1. ��־�ļ���������ʽ;
    2. ÿ1mb����һ��log�ļ�;
    3. ����ʱ��������log�ļ�;
    4. log����ĸ�ʽ;
* ����ͨ���ú���ע����sink, ÿһ��sink�����Խ��ܴ�����־, ��֮���໥����;

### ���������ʽ
```C++
void Init()
{
    typedef sinks::synchronous_sink< sinks::text_ostream_backend > text_sink;
    //����������ļ�complex.log��
	boost::shared_ptr< text_sink > sink = boost::make_shared<text_sink>();
	sink->locked_backend()->add_stream(boost::make_shared<std::ofstream>("complex.log"));

	//ͬʱ�����console��
	sink->locked_backend()->add_stream(boost::make_shared<std::ostream>(std::cout.rdbuf()));
	//�����console��������ȼ�
	boost::shared_ptr<std::ostream> stream_ptr(&std::clog, boost::null_deleter());
	sink->locked_backend()->add_stream(stream_ptr);
	logging::core::get()->add_sink(sink);
}
```
���ַ�ʽ, �ͷ�ʽ1��Ч����ȫһ��; �����Ҫ����һ��sink, ��������ϸ�Ľ���;
* sink ��Ϊǰ��(frontend)�ͺ��(backend)sink;
* frontend sink(����������synchronous_sink��ģ��)����sink��ͨ������; ��: �߳�ͬ��ģʽ, ����, ��ʽ����;
* ���(������������ʹ��text_ostream_backend��)ʵ��sink�ض������飬��д��һ���ļ���
* boost log���ṩ������ֱ��ʹ�õ�frontend sinks and backend sinks��    
* synchronous_sink ����sink���̰߳�ȫ��, ��backend sink���õ����̰߳�ȫ����;
* text_ostream_backend ����Խ���ʽ����logд�뵽����stream��; ����ֱ�չ����������ļ�, �Լ����������console�ķ�ʽ;
* ע����sink��ע��һ��sink���������Ŀ��stream���������; ����ǰ��sink֮���໥����; ����Ч�ʽϸ�;
* locked_backend��Ա�������̰߳�ȫ��; ����������һ��ָ��backend������ָ��; **ע��:** ������ָ������ڼ�, backendһֱ����ס״̬; Ҳ����ζ�������߳���Ҫ����־д����backend sink�Ͻ��ᱻ����, ֱ��backend sink���ͷ�;

boost log���ṩ�˶�����ڴ��������־�߼���backends sinks; ����: ����Խ���־����(by network)��syslog ������ͨ��ָ��syslog backend; ��Ҳ��������Windows NT Event log backend�������Ӧ�õ�����ʱ��; �ȵ�;

## ��־������д��
�������Ǽ�˵��һ����־��δ洢�Լ��洢�����������; �������˽�һ����δ�����־Դ�Լ����д��־ ?
```
src::logger lg;
```
1. ��־Դsources����Ҫ��sinksһ����ע��, sourcesֱ�Ӻ�log coreֱ�ӽ���;
2. boost log �ṩ�����汾��logger, �̰߳�ȫ��ͷ��̰߳�ȫ��;
3. boost log���ṩ����������ͬ������loggers; ����־�ȼ��Լ���channel֧��; ��Щ�������������໥��Ϲ��ɲ�ͬ���ܵ�logger;

### ȫ��logger����
log���ṩ��һ������ȫ��loggers����ķ���:

```
BOOST_LOG_INLINE_GLOBAL_LOGGER_DEFAULT(global_logger, src::logger_mt)
```
˵��:
1. global_logger �û��Զ����logger; ��������ʹ�ø�global_logger��ȡloggerʵ��;
>     //��ȡȫ��Ψһ��loggerʵ��
>     src::logger_mt& lg = my_logger::get();
2. logger_mt �̰߳�ȫlogger;

### д��־

������ʹ�õ�����һ��logger(���Ա����ģʽ����ȫ��ģʽ, �̰߳�ȫ�򲻰�ȫ), ������ʹ��һ�·�ʽд��־:
```
//����ȫ��logger ����
BOOST_LOG_INLINE_GLOBAL_LOGGER_DEFAULT(global_logger, src::logger_mt);

void PutLogRecords()
{
	//��ȡloggerʵ��
	src::logger_mt& lg = global_logger::get();
	BOOST_LOG(lg) << "global macro write log records";
	//logд��
	logging::record rec = lg.open_record();
	if (rec)
	{
		logging::record_ostream strm(rec);
		strm << "global logger write";
		strm.flush();
		lg.push_record(boost::move(rec));
	}
}
```
���:
```
[2017-04-19 15:38:01.191468] [0x00007ffff7fd8740] [info]    global macro write log records
[2017-04-19 15:38:01.191551] [0x00007ffff7fd8740] [info]    global logger write
```
˵��:
1. ����ʹ��BOOST_LOG�������־д��;
2. open_record���ÿ��������ж�record�Ƿ񱻴����Լ����ٱ�һ��sinks����;
3. ����Ҳ����������׶�����;

## ��־����
* ÿһ��log record���ж������������, ��Щ���Դ�������־��¼������: �ڴ����е�λ��, ִ��ģ�������, ��ǰʱ��, ���������������ص��κ����ݺ����л���.
* ÿһ�����Զ����Ӧһ������ֵ, ���Թ�����, ��ʽ����sinksʹ��; boost log�����Դ������û����õ�����.
* ���Է�Ϊ����:��source-specific, thread-specific �� global������log record�γ�, ��֮������������һ�����ϴ��ݵ�sinks��;
* ÿһ��������Ҫһ����һ�޶�������, �������; ����ڲ�ͬ�����������ͬ������, ���Ǵ���: Դ������� > �߳�������� > ȫ������; ��������ͨ��ע�᱾������������ȫ�ֺ��߳��������, �Լ����̸߳���;

### ����ע��
#### ��������
һЩ�������κγ��򶼻�ʹ�õ���, ����: ��־��¼����, ʱ��ȵ�; ��Щ���õ����Կ���ͨ������ĺ������:
```
logging::add_common_attributes();
```
ͨ���ú�������, ��־��ID, ʱ���, ����ID, �߳�ID���ᱻע��; ��Ȼ, ���߳�ģʽ��, �߳�IDû�б�ע��.

**ע��: **
1. ��������Ĭ��, û��ע���κ�����, �����ڳ����ʼ��ʱ�������ע��; ֮ǰTrivial loggingʵ������ʹ�õ�Ĭ��sink, ���԰�����־�ȼ���Ĭ������, ��˲���Ҫ��ʼ��;
2. һ����ʹ�ù���, ��ʽ��, �Լ���Ĭ��sink, ��ͱ���ע���Ҫ������.


����һЩ������logger����ʱ��ᱻ�Զ�ע��, ����: ��־�ĵȼ�.

#### ��������
```
//�Զ���log��ȫ����
void logging_function()
{
	src::severity_logger< severity_level > slg;
	BOOST_LOG_SEV(slg, normal) << "A regular message";
	BOOST_LOG_SEV(slg, warning) << "Something bad is going on but I can handle it";
	BOOST_LOG_SEV(slg, critical) << "Everything crumbles, shoot me now!";
}

//���scope name ��ÿһ��log record
void named_scope_logging()
{
	BOOST_LOG_NAMED_SCOPE("named_scope_logging");
	src::severity_logger< severity_level > slg;
	BOOST_LOG_SEV(slg, normal) << "Hello from the function named_scope_logging!";
}

//����Զ���tag
void tagged_logging()
{
	src::severity_logger< severity_level > slg;
	slg.add_attribute("Tag", attrs::constant< std::string >("My tag value"));
	BOOST_LOG_SEV(slg, normal) << "Here goes the tagged record";
}
```
˵��:
1. ͨ������Զ���tag����, �������ں����ĸ�ʽ���Լ�����;
2. ���scope name���Ա��Ӧ�ó���ͬ���ֲ���log;

����, Ҳ����ʹ��boost log���Timeline����, ����һ���򵥵�profiling�������������, Timeline�����߾��ȵ�ʱ����Ϣ�����������ж���һ���ֳ�����Ҫ�ܶ��ִ��ʱ�䡣����:
```
void timed_logging()
{
	BOOST_LOG_SCOPED_THREAD_ATTR("Timeline", attrs::timer());
	src::severity_logger< severity_level > slg;
	BOOST_LOG_SEV(slg, normal) << "Starting to time nested functions";
	logging_function();
	BOOST_LOG_SEV(slg, normal) << "Stopping to time nested functions";
}
```
���:
```
00000003: <normal>	() [00:00:00.000005] Starting to time nested functions
00000004: <normal>	() [00:00:00.000176] A regular message
00000005: <warning>	() [00:00:00.000230] Something bad is going on but I can handle it
00000006: <critical>() [00:00:00.000267] Everything crumbles, shoot me now!
00000007: <normal>	() [00:00:00.000301] Stopping to time nested functions
```
### ����ռλ��
����ռλ����Ҫ�ڽ������Ĳ��ֻ��õ�,  ÿһ������ָ��һ���ؼ���, ����ؼ�������־���˺͸�ʽ���лᱻ�õ�;
```
//placeholder
BOOST_LOG_ATTRIBUTE_KEYWORD(line_id, "LineID", unsigned int)
BOOST_LOG_ATTRIBUTE_KEYWORD(severity, "Severity", severity_level)
BOOST_LOG_ATTRIBUTE_KEYWORD(tag_attr, "Tag", std::string)
BOOST_LOG_ATTRIBUTE_KEYWORD(scope, "Scope", attrs::named_scope::value_type)
BOOST_LOG_ATTRIBUTE_KEYWORD(timeline, "Timeline", attrs::timer::value_type)
```
˵��:
1. ��ĵ�һ����������keyword, �ڶ���������������, ��������������ֵ����;

## ��־��ʽ��
���֮ǰ��ʵ��(������ʹ��Ĭ�����������), û����Ӹ�ʽ��, �������־��Ϣ��ֻ��Message��Ϣ;

���÷���:
1. `add_file_log`��ͨ��`keyword::format`����;
2. ����ֶ�������sink, ����ͨ��sink��set_formatter������;

### lambda ����ʽ��
```
void init()
{
    logging::add_file_log
    (
        keywords::file_name = "sample_%N.log",
        // ��־�����ʽ
        // 1: <normal> A normal severity message
        keywords::format =
        (
            expr::stream
                << expr::attr< unsigned int >("LineID")
                << ": <" << logging::trivial::severity
                << "> " << expr::smessage
        )
    );
}
```
����, �����ʽ��ʹ�õĶ���ռλ��, ���ǿ����ڹ������͸�ʽ��������ʹ��ͬ����ռλ��; attrռλ����severityռλ������, ���Ǳ�ʾ����ֵ; ��ͬ����severityռλ����ʾ����Ϊ��Severity��������Ϊ
trivial::severity_level�����ԣ�attr���Ա�ʾ�κ����ԡ�
����Ҳ����ʹ�����淽ʽ����severityռλ��:
```
expr::attr< logging::trivial::severity_level >("Severity")
```

Ҳ�����ṩһЩ�������Եĳ�ʼ��������, ʱ��, LineID�ȵ�;
```
sink->set_formatter
		(
		expr::stream
		<< std::hex << std::setw(8) << std::setfill('0') << line_id << std::dec << std::setfill(' ')
		<< ": <" << severity << ">\t"
		<< "(" << scope << ") "
		<< expr::if_(expr::has_attr(tag_attr))
		[
			expr::stream << "[" << tag_attr << "] "
		]
	<< expr::if_(expr::has_attr(timeline))
		[
			expr::stream << "[" << timeline << "] "
		]
	<< expr::smessage
		);
```

### Boost.Format ����ʽ��
```
void init()
{
    typedef sinks::synchronous_sink< sinks::text_ostream_backend > text_sink;
    boost::shared_ptr< text_sink > sink = boost::make_shared< text_sink >();
    sink->locked_backend()->add_stream(boost::make_shared< std::ofstream >("sample.log"));
    // ��־�����ʽ
    // 1: <normal> A normal severity message
    // 2: <error> An error severity message
    sink->set_formatter
    (
        expr::format("%1%: <%2%> %3%")
            % expr::attr< unsigned int >("LineID")
            % logging::trivial::severity
            % expr::smessage
    );

    logging::core::get()->add_sink(sink);
}
```
boost log��ʽ��Ҳ֧��[Boost.Format](http://www.boost.org/doc/libs/1_63_0/libs/log/doc/html/log/tutorial/formatters.html)��ʽ, ������﷨��������׸��;

### ר�ŵĸ�ʽ��
boost log���ṩ��ר�ŵĳ�ʼ��Ϊ��Щ��������, ������, ʱ���Լ�����������; ����: ��ʽ��Date��timeΪ[Boost.DateTime](http://www.boost.org/doc/libs/1_64_0/doc/html/date_time.html)��ʽ.
```
void init()
{
    logging::add_file_log
    (
        keywords::file_name = "sample_%N.log",
        // YYYY-MM-DD HH:MI:SS: <normal> A normal severity message
        keywords::format =
        (
            expr::stream
                << expr::format_date_time< boost::posix_time::ptime >("TimeStamp", "%Y-%m-%d %H:%M:%S")
                << ": <" << logging::trivial::severity
                << "> " << expr::smessage
        )
    );
}
```
���ֶ�����ʱ��ĸ�ʽ��Ҳ����������Boost.Format���һ��ʹ��.

### �ַ�������ʽ��
��Щ�����, ���ǿ���ʹ���ַ���ģ���ʽ����־, �ַ�����ʹ��`%%`����ʾռλ��, ��: `%Message%`ռλ���ᱻ��־��¼message������;
```
void init()
{
    logging::add_file_log
    (
        keywords::file_name = "sample_%N.log",
        keywords::format = "[%TimeStamp%]: %Message%"
    );
}
```
ע: sink�ĳ�Ա����`set_formatter`��֧�ָ÷���ʽ��(�ַ������);

### �Զ����ʽ������
���漸�ָ�ʽ����ʽ, ���Ƕ���ͨ���������ݸ�����ĸ�ʽ������; ʵ����, ����Ҳ���Զ��������Լ���ʽ������, Ȼ��ͨ����������ʽ���ݸ�log���ʽ������!

```
void MyFormatter(logging::record_view const& rec, logging::formatting_ostream& strm)
{
    strm << logging::extract< unsigned int >("LineID", rec) << ": ";
    strm << "<" << rec[logging::trivial::severity] << "> ";
    strm << rec[expr::smessage];
}

void init()
{
    typedef sinks::synchronous_sink< sinks::text_ostream_backend > text_sink;
    boost::shared_ptr< text_sink > sink = boost::make_shared< text_sink >();
    sink->locked_backend()->add_stream(boost::make_shared< std::ofstream >("sample.log"));
    sink->set_formatter(&MyFormatter);
    logging::core::get()->add_sink(sink);
}
```
˵��:
* logging::record_view ��log::record����; ����: record_view�ǲ����޸���ʵ��ǳ����; ��־��ʽ���͹������в���������record_view�����, ������ֱ��Ӱ�쵽log record, ��Щrecord���ܱ�����sinksʹ���������߳���;

## ��־����
����ܶ�ζ��ᵽ��־����, �ڸ����м�ʹ�ù���־�ȼ�����; �ϸ��ӵ���־������������:
```
BOOST_LOG_ATTRIBUTE_KEYWORD(line_id, "LineID", unsigned int)
BOOST_LOG_ATTRIBUTE_KEYWORD(severity, "Severity", severity_level)
BOOST_LOG_ATTRIBUTE_KEYWORD(tag_attr, "Tag", std::string)

void init()
{
    logging::formatter fmt = expr::stream
        << std::setw(6) << std::setfill('0') << line_id << std::setfill(' ')
        << ": <" << severity << ">\t"
        << expr::if_(expr::has_attr(tag_attr))
           [
               expr::stream << "[" << tag_attr << "] "
           ]
        << expr::smessage;

    // Initialize sinks
    typedef sinks::synchronous_sink< sinks::text_ostream_backend > text_sink;
    boost::shared_ptr< text_sink > sink = boost::make_shared< text_sink >();
    sink->locked_backend()->add_stream(boost::make_shared< std::ofstream >("full.log"));
    sink->set_formatter(fmt);

    logging::core::get()->add_sink(sink);
    sink = boost::make_shared< text_sink >();

    sink->locked_backend()->add_stream(boost::make_shared< std::ofstream >("important.log"));
    sink->set_formatter(fmt);
    sink->set_filter(severity >= warning || (expr::has_attr(tag_attr) && tag_attr == "IMPORTANT_MESSAGE"));
    logging::core::get()->add_sink(sink);
    logging::add_common_attributes();
}
```
ʵ�����������־�ļ�"important.log"��sink��ӹ���; ͨ�����ú���`set_filter`ʵ��, �������ղ�����Ȼ��Boost.Phoenix���;

��Ȼ��Ҳ�������Զ���ʽ������һ��, �Զ�����˺���:
```
bool MyFilter(logging::value_ref< severity_level, tag::severity > const& level, logging::value_ref< std::string, tag::tag_attr > const& tag)
{
    return level >= warning || tag == "IMPORTANT_MESSAGE";
}

void init()
{
    namespace phoenix = boost::phoenix;
    sink->set_filter(phoenix::bind(&my_filter, severity.or_none(), tag_attr.or_none()));.
}
```
# ���

## ��־��¼(Records)
��־��¼�������е���־��ص���Ϣ; ��Щ��Ϣ(������־���ݱ���)��ʹ������������ֵ��ʾ; ��Щ����ֵ���Ա�����, ��ʽ��, sinks�ȹ���ʹ��;  
�ر������ֵ����ͨ�����¼��ַ�������:
```
struct PrintVisitor
{
	typedef void ResultValue;
	ResultValue operator()(SeverityLevel level) const
	{
		std::cout << "PrintVisitor: " << level << std::endl;
	}
};

//ʹ��logging::visit
void PrintSeverityVisit(const logging::record& rec)
{
    //ʹ��lambda����PrintVisitor
	logging::visit<SeverityLevel>("Severity", rec,
		[=](SeverityLevel level) { std::cout << "PrintSeverityVisit Lambda: "<< level << std::endl; });
	logging::visit<SeverityLevel>("Severity", rec, PrintVisitor());
}

//ʹ��logging::extract
void PrintSeverityExtract(const logging::record& rec)
{
	logging::value_ref<SeverityLevel> level = logging::extract<SeverityLevel>("Severity", rec);
	std::cout << "PrintSeverityExtract: " << level << std::endl;
}

//ֱ��ͨ���±�
void PrintSeveritySubscript(const logging::record& rec)
{
	logging::value_ref<SeverityLevel, tag::severity> level = rec[severity];
	std::cout << "PrintSeveritySubscript: " << level << std::endl;
	logging::value_ref<unsigned int, tag::line_id> line = rec[line_id];
	std::cout << "PrintSeveritySubscript: " << line << std::endl;
}

//look up
void PrintSeverityLookup(const logging::record& rec)
{
	const logging::attribute_value_set& values = rec.attribute_values();
	auto iter = values.find("Severity");
	if (iter != values.end())
	{
		const logging::attribute_value& value = iter->second;
		std::cout << value.extract<SeverityLevel>() << std::endl;
	}
}
```
