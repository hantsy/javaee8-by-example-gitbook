# Example Codes

You can get the sample codes of this book from my GitHub account, and install JDK 8, NetBeans 8.2 or 9, Glassfish v5, and play the sample codes yourself.

### Get source codes

The source codes of this book is available on [https://github.com/hantsy/ee8-sandbox](https://github.com/hantsy/ee8-sandbox).

Clone it into your local disk.

```text
git clone https://github.com/hantsy/ee8-sandbox
```

### Install Oracle JDK 8

1. Get the [latest Oracle Java 8](http://java.oracle.com), alternatively you can also get a copy of [AdoptOpenJDK](https://adoptopenjdk.net/) instead.
2. Follow [the official installation guide](https://docs.oracle.com/javase/8/docs/technotes/guides/install/install_overview.html) to install it into your local system.
3. To verify the installation, open a terminal and type `java --version`, it should print java version info.

### Install NetBeans IDE

At the moment I wrote down the book, there is no more  IDEs support Java EE 8.

1. Download NetBeans from [http://www.netbeans.org](http://www.netbeans.org),  NetBeans 8.2+ is required.
2. Install it into your local system.

>NetBeans is an Apache project now, check https://netbeans.apache.org

### Install Glassfish v5

1. Download Glassfish v5 from [http://github.com/glassfish](http://github.com/glassfish)
2. Extract the archive into your local disc.
3. Open NetBeans IDE, click **Services** tab. If it is not opened, open it from **Window** menu.
4. Right click the **Servers** node, in the popup context menu, click **Add Server** and follow the wizard to add Glassfish v5 into your IDE.

### Import sample codes

1. Starts up NetBeans IDE.
2. Click **Open** icon or **File**/**Open** menu to select a project. Maven projects are recognized by NetBeans IDE automatically.
3. Select the above cloned project, open it in NetBeans IDE, it looks like.

![nb](../.gitbook/assets/nb-javaee8.png)

Click any sub project node under **Modules** to open the certain project, and run it on Glassfish application server by click **Run** in the context menu.



