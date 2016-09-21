## gradle task脚本删除log日志
  
正在学习gradle,所以不多说直接上代码

在`build.gradle`中加入以下task，然后在Android Studio 的`Terminal`中执行，windows输入`gradlew -q dl`,mac `gradle -q dl`，等一会你就发现所有`Log.`的代码被删除。

		  task dl << {

		  def rootPath = project.projectDir.absolutePath;

		  def files = new File(rootPath + "/src");

		  println files.getAbsolutePath()

		  deleteLog(files)
		}

		void deleteLog(File listFiles) {
		  if (listFiles != null) {
		    listFiles.eachFile {File file ->
		      if (file != null) {
		        if (file.isFile()) {
		          if (file.canRead() && file.name.endsWith(LogConstant.suffix)) {
		            println file.getAbsolutePath()

		            deleteFileLog(file);
		          }
		        } else if (file.isDirectory()) {
		          deleteLog(file);
		        }
		      }
		    }
		  }
		}

--

		void deleteFileLog(File javafile) {
		  def endFlag = 0;
		  File ftmp = file(javafile.getAbsolutePath() + ".tmp");
		  def printWriter = ftmp.newPrintWriter(LogConstant.charset);
		  def reader = javafile.newReader(LogConstant.charset);
		  def tmpline = null;
		  String line;
		  while ((line = reader.readLine()) != null) {
		    if (line != null) {
		      tmpline = line.trim();
		      if (tmpline.startsWith(LogConstant.head) || endFlag == 1) {

		        if (tmpline.endsWith(LogConstant.tail_end)) {
		          endFlag = 0;
		          printWriter.write(";\n")
		          continue
		        } else {
		          endFlag = 1;
		          continue
		        }
		      } else {
		        printWriter.write(line + "\n");
		      }
		    }
		  }

		  reader.close();

		  printWriter.flush();
		  printWriter.close();

		  javafile.delete();
		  ftmp.renameTo(javafile.getAbsolutePath());
		}

--

		task LogConstant {
		  ext.head = 'Log.'
		  ext.tail_end = ');'
		  ext.suffix = '.java'
		  ext.charset = "utf-8"
		}