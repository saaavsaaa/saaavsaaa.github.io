org.thymeleaf.exceptions.TemplateInputException: Error resolving template "index", template might not exist or might not be accessible by any of the configured Template Resolvers
	at org.thymeleaf.TemplateRepository.getTemplate(TemplateRepository.java:246)
	at org.thymeleaf.TemplateEngine.process(TemplateEngine.java:1104)
	at org.thymeleaf.TemplateEngine.process(TemplateEngine.java:1060)
	at org.thymeleaf.TemplateEngine.process(TemplateEngine.java:1011)

有一个页面在别人机器上都不出问题，在我机器上跑报这个错，由于相关代码我并没有改过，所以怀疑是环境问题，在final class TemplateRepository:getTemplate:
templateInputStream = reader.getResourceAsStream(templateProcessingParameters, characterEncoding)这句代码上打了断点，发现路径都对，
但是返回值是null，于是怀疑大小写问题，果然，唯独这个出错页面命名是用了大写开头，使用的时候用的是小写Index --> index，而windows不区分，
幸好我本地开发环境是linux，不然估计到上线可能都发现不了。
另外，系统环境还有一种问题，https://stackoverflow.com/questions/7858987/apache-2-throws-no-such-file-or-directory-exec-of-usr-lib-cgi-bin-fst-cgi/38891793#38891793
可能的一种原因是文件格式
