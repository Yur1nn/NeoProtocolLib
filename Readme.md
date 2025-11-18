# ProtocolLib

> **⚠️ Project Status: On Hiatus**
> 
> This fork (NeoProtocolLib) is currently on hiatus due to the original ProtocolLib developers having resumed active development on the project. This fork will be continued when the ProtocolLib developers are no longer actively maintaining the original project.

Certain tasks are impossible to perform with the standard Bukkit API, and may require
working with and even modifying Minecraft directly. A common technique is to modify incoming
and outgoing packets, or inject custom packets into the
stream. This is quite cumbersome to do, however, and most implementations will break
as soon as a new version of Minecraft has been released, mostly due to obfuscation.

Critically, different plugins that use this approach may _hook_ into the same classes,
with unpredictable outcomes. More than often this causes plugins to crash, but it may also
lead to more subtle bugs.

### Resources

* [Spigot Page](https://spigotmc.org/resources/protocollib.1997/)
* [Releases](https://github.com/dmulloy2/ProtocolLib/releases)
* [Dev Build](https://github.com/dmulloy2/ProtocolLib/releases/tag/dev-build)
* [JavaDoc](https://dmulloy2.net/ProtocolLib/javadoc/)
* [Protocol Wiki](https://minecraft.wiki/w/Minecraft_Wiki:Projects/wiki.vg_merge/Protocol)

### Compilation

ProtocolLib is built with [Gradle](https://gradle.org/). If you have it installed, just run
`./gradlew build` in the root project folder. Other gradle targets you may be interested in 
include `clean`, `test`, and `shadowJar`. `shadowJar` will create a jar with all dependencies
(ByteBuddy) included.

### A new API

__ProtocolLib__ attempts to solve this problem by providing an event API, much like Bukkit,
that allows plugins to monitor, modify, or cancel packets sent and received. But, more importantly,
the API also hides all the gritty, obfuscated classes with a simple index based read/write system.
You no longer have to reference CraftBukkit!

### Using ProtocolLib

To use this library, first add ProtocolLib.jar to your Java build path. Then, add ProtocolLib
as a dependency or soft dependency to your plugin.yml file like any other plugin:

````yml
depend: [ ProtocolLib ]
````

You can also add ProtocolLib as a Maven dependency:

````xml
<dependencies>
  <dependency>
    <groupId>net.dmulloy2</groupId>
    <artifactId>ProtocolLib</artifactId>
    <version>5.4.0</version>
    <scope>provided</scope>
  </dependency>
</dependencies>
````

Or use the maven dependency with gradle:

```gradle
repositories {
    mavenCentral()
}

dependencies {
    compileOnly 'net.dmulloy2:ProtocolLib:5.4.0'
}
```

Then get a reference to ProtocolManager in onLoad() or onEnable() and you're good to go.

````java
private ProtocolManager protocolManager;

public void onLoad() {
    protocolManager = ProtocolLibrary.getProtocolManager();
}
````

To listen for packets sent by the server to a client, add a server-side listener:

````java
// Disable all sound effects
protocolManager.addPacketListener(new PacketAdapter(
    this,
    ListenerPriority.NORMAL,
    PacketType.Play.Server.NAMED_SOUND_EFFECT
) {
    @Override
    public void onPacketSending(PacketEvent event) {
        event.setCancelled(true);
    }
});
````

It's also possible to read and modify the content of these packets. For instance, you can create a global
censor by listening for Packet3Chat events:

````java
// Censor
protocolManager.addPacketListener(new PacketAdapter(
    this,
    ListenerPriority.NORMAL,
    PacketType.Play.Client.CHAT
) {
    @Override
    public void onPacketReceiving(PacketEvent event) {
        PacketContainer packet = event.getPacket();
        String message = packet.getStrings().read(0);

        if (message.contains("shit") || message.contains("damn")) {
            event.setCancelled(true);
            event.getPlayer().sendMessage("Bad manners!");
        }
    }
});
````

### Sending packets

Normally, you might have to do something ugly like the following:

````java
PacketPlayOutExplosion fakeExplosion = new PacketPlayOutExplosion(
    player.getLocation().getX(),
    player.getLocation().getY(),
    player.getLocation().getZ(),
    3.0F,
    new ArrayList<>(),
    new Vec3D(
        player.getVelocity().getX() + 1,
        player.getVelocity().getY() + 1,
        player.getVelocity().getZ() + 1
    )
);

((CraftPlayer) player).getHandle().b.a(fakeExplosion);
````

But with ProtocolLib, you can turn that into something more manageable:

````java
PacketContainer fakeExplosion = new PacketContainer(PacketType.Play.Server.EXPLOSION);
fakeExplosion.getDoubles()
    .write(0, player.getLocation().getX())
    .write(1, player.getLocation().getY())
    .write(2, player.getLocation().getZ());
fakeExplosion.getFloat().write(0, 3.0F);
fakeExplosion.getBlockPositionCollectionModifier().write(0, new ArrayList<>());
fakeExplosion.getVectors().write(0, player.getVelocity().add(new Vector(1, 1, 1)));

protocolManager.sendServerPacket(player, fakeExplosion);
````

### Compatibility

One of the main goals of this project was to achieve maximum compatibility with CraftBukkit. And the end
result is quite flexible. It's likely that I won't have to update ProtocolLib for anything but bug fixes
and new features.

How is this possible? It all comes down to reflection in the end. Essentially, no name is hard coded -
every field, method and class is deduced by looking at field types, package names or parameter
types. It's remarkably consistent across different versions.

---

# ProtocolLib (Tiếng Việt)

> **⚠️ Trạng thái dự án: Tạm dừng**
> 
> Fork này (NeoProtocolLib) hiện đang tạm dừng do các nhà phát triển ProtocolLib gốc đã tiếp tục phát triển tích cực dự án. Fork này sẽ được tiếp tục khi các nhà phát triển ProtocolLib không còn bảo trì tích cực dự án gốc nữa.

Một số tác vụ không thể thực hiện được với Bukkit API tiêu chuẩn, và có thể yêu cầu
làm việc hoặc thậm chí chỉnh sửa Minecraft trực tiếp. Một kỹ thuật phổ biến là chỉnh sửa các
gói tin đến và đi, hoặc chèn các gói tin tùy chỉnh vào luồng dữ liệu. Tuy nhiên, điều này khá
phức tạp để thực hiện, và hầu hết các triển khai sẽ bị hỏng ngay khi một phiên bản mới của
Minecraft được phát hành, chủ yếu do việc làm rối mã.

Quan trọng là, các plugin khác nhau sử dụng phương pháp này có thể _móc_ vào cùng các lớp,
với kết quả không thể đoán trước. Thường xuyên hơn, điều này khiến các plugin bị sập, nhưng
cũng có thể dẫn đến các lỗi tinh vi hơn.

### Tài nguyên

* [Trang Spigot](https://spigotmc.org/resources/protocollib.1997/)
* [Bản phát hành](https://github.com/dmulloy2/ProtocolLib/releases)
* [Bản dựng Dev](https://github.com/dmulloy2/ProtocolLib/releases/tag/dev-build)
* [JavaDoc](https://dmulloy2.net/ProtocolLib/javadoc/)
* [Wiki Protocol](https://minecraft.wiki/w/Minecraft_Wiki:Projects/wiki.vg_merge/Protocol)

### Biên dịch

ProtocolLib được xây dựng với [Gradle](https://gradle.org/). Nếu bạn đã cài đặt nó, chỉ cần chạy
`./gradlew build` trong thư mục gốc của dự án. Các mục tiêu gradle khác bạn có thể quan tâm
bao gồm `clean`, `test`, và `shadowJar`. `shadowJar` sẽ tạo một jar với tất cả các phụ thuộc
(ByteBuddy) được bao gồm.

### Một API mới

__ProtocolLib__ cố gắng giải quyết vấn đề này bằng cách cung cấp một API sự kiện, giống như Bukkit,
cho phép các plugin theo dõi, chỉnh sửa hoặc hủy các gói tin được gửi và nhận. Nhưng, quan trọng hơn,
API cũng ẩn tất cả các lớp phức tạp, bị làm rối với một hệ thống đọc/ghi dựa trên chỉ mục đơn giản.
Bạn không còn phải tham chiếu đến CraftBukkit nữa!

### Sử dụng ProtocolLib

Để sử dụng thư viện này, trước tiên hãy thêm ProtocolLib.jar vào đường dẫn build Java của bạn. Sau đó, thêm ProtocolLib
như một phụ thuộc hoặc phụ thuộc mềm vào file plugin.yml của bạn giống như bất kỳ plugin nào khác:

````yml
depend: [ ProtocolLib ]
````

Bạn cũng có thể thêm ProtocolLib như một phụ thuộc Maven:

````xml
<dependencies>
  <dependency>
    <groupId>net.dmulloy2</groupId>
    <artifactId>ProtocolLib</artifactId>
    <version>5.4.0</version>
    <scope>provided</scope>
  </dependency>
</dependencies>
````

Hoặc sử dụng phụ thuộc maven với gradle:

```gradle
repositories {
    mavenCentral()
}

dependencies {
    compileOnly 'net.dmulloy2:ProtocolLib:5.4.0'
}
```

Sau đó lấy một tham chiếu đến ProtocolManager trong onLoad() hoặc onEnable() và bạn đã sẵn sàng.

````java
private ProtocolManager protocolManager;

public void onLoad() {
    protocolManager = ProtocolLibrary.getProtocolManager();
}
````

Để lắng nghe các gói tin được gửi bởi máy chủ đến một client, thêm một trình lắng nghe phía máy chủ:

````java
// Tắt tất cả hiệu ứng âm thanh
protocolManager.addPacketListener(new PacketAdapter(
    this,
    ListenerPriority.NORMAL,
    PacketType.Play.Server.NAMED_SOUND_EFFECT
) {
    @Override
    public void onPacketSending(PacketEvent event) {
        event.setCancelled(true);
    }
});
````

Cũng có thể đọc và chỉnh sửa nội dung của các gói tin này. Ví dụ, bạn có thể tạo một
bộ lọc toàn cục bằng cách lắng nghe các sự kiện Packet3Chat:

````java
// Bộ lọc
protocolManager.addPacketListener(new PacketAdapter(
    this,
    ListenerPriority.NORMAL,
    PacketType.Play.Client.CHAT
) {
    @Override
    public void onPacketReceiving(PacketEvent event) {
        PacketContainer packet = event.getPacket();
        String message = packet.getStrings().read(0);

        if (message.contains("shit") || message.contains("damn")) {
            event.setCancelled(true);
            event.getPlayer().sendMessage("Bad manners!");
        }
    }
});
````

### Gửi gói tin

Thông thường, bạn có thể phải làm điều gì đó xấu xí như sau:

````java
PacketPlayOutExplosion fakeExplosion = new PacketPlayOutExplosion(
    player.getLocation().getX(),
    player.getLocation().getY(),
    player.getLocation().getZ(),
    3.0F,
    new ArrayList<>(),
    new Vec3D(
        player.getVelocity().getX() + 1,
        player.getVelocity().getY() + 1,
        player.getVelocity().getZ() + 1
    )
);

((CraftPlayer) player).getHandle().b.a(fakeExplosion);
````

Nhưng với ProtocolLib, bạn có thể biến điều đó thành thứ gì đó dễ quản lý hơn:

````java
PacketContainer fakeExplosion = new PacketContainer(PacketType.Play.Server.EXPLOSION);
fakeExplosion.getDoubles()
    .write(0, player.getLocation().getX())
    .write(1, player.getLocation().getY())
    .write(2, player.getLocation().getZ());
fakeExplosion.getFloat().write(0, 3.0F);
fakeExplosion.getBlockPositionCollectionModifier().write(0, new ArrayList<>());
fakeExplosion.getVectors().write(0, player.getVelocity().add(new Vector(1, 1, 1)));

protocolManager.sendServerPacket(player, fakeExplosion);
````

### Tương thích

Một trong những mục tiêu chính của dự án này là đạt được khả năng tương thích tối đa với CraftBukkit. Và kết quả
cuối cùng khá linh hoạt. Có khả năng tôi sẽ không phải cập nhật ProtocolLib cho bất cứ điều gì ngoài việc sửa lỗi
và các tính năng mới.

Làm thế nào điều này có thể? Cuối cùng, tất cả đều quy về reflection. Về cơ bản, không có tên nào được mã hóa cứng -
mọi trường, phương thức và lớp đều được suy luận bằng cách xem xét các kiểu trường, tên gói hoặc kiểu tham số.
Nó đáng chú ý là nhất quán trên các phiên bản khác nhau.
