# Created by: Dmitry Marakasov <amdmi3@FreeBSD.org>
# $FreeBSD: head/games/freeminer/Makefile 393445 2015-08-02 21:07:38Z amdmi3 $

PORTNAME=	freeminer
PORTVERSION=	0.4.13.7
CATEGORIES=	games
MASTER_SITES=	https://github.com/${PORTNAME}/${PORTNAME}/releases/download/${PORTVERSION}/

MAINTAINER=	amdmi3@FreeBSD.org
COMMENT=	Open source sandbox game inspired by Minecraft

LICENSE=	LGPL21 GPLv3 # or later
LICENSE_COMB=	multi
LICENSE_FILE_GPLv3=	${WRKSRC}/COPYING

LIB_DEPENDS=	libmsgpack.so:${PORTSDIR}/devel/msgpack \
		libfreetype.so:${PORTSDIR}/print/freetype2

USES=		compiler:c++11-lib cmake zip
CMAKE_ARGS=	-DCUSTOM_MANDIR="${PREFIX}/man" \
		-DCUSTOM_EXAMPLE_CONF_DIR="${PREFIX}/etc" \
		-DRUN_IN_PLACE=0 \
		-DENABLE_SYSTEM_MSGPACK=1

PORTDATA=	*
PORTDOCS=	*

OPTIONS_DEFINE=	CLIENT SERVER THREADS CURL SOUND LEVELDB SQLITE3 REDIS LUAJIT MINETEST NLS DOCS EXAMPLES
OPTIONS_DEFAULT=CLIENT SERVER THREADS CURL SOUND LEVELDB LUAJIT
OPTIONS_SUB=	yes

CLIENT_DESC=	Build client
SERVER_DESC=	Build server
CURL_DESC=	Enable cURL support for fetching media
SOUND_DESC=	Enable sound
LEVELDB_DESC=	Enable LevelDB backend
LUAJIT_DESC=	LuaJIT support
MINETEST_DESC=	Use minetest protocol (slow and buggy)

CLIENT_CMAKE_ON=	-DBUILD_CLIENT=1
CLIENT_CMAKE_OFF=	-DBUILD_CLIENT=0
CLIENT_LIB_DEPENDS=	libpng.so:${PORTSDIR}/graphics/png \
			libIrrlicht.so:${PORTSDIR}/x11-toolkits/irrlicht
CLIENT_USES=		jpeg
CLIENT_USE=		GL=gl XORG=x11
SERVER_CMAKE_ON=	-DBUILD_SERVER=1
SERVER_CMAKE_OFF=	-DBUILD_SERVER=0
CURL_CMAKE_ON=		-DENABLE_CURL=1
CURL_CMAKE_OFF=		-DENABLE_CURL=0
CURL_LIB_DEPENDS=	libcurl.so:${PORTSDIR}/ftp/curl
NLS_CMAKE_ON=		-DENABLE_GETTEXT=1
NLS_CMAKE_OFF=		-DENABLE_GETTEXT=0
NLS_USES=		gettext
NLS_LDFLAGS=		-L${LOCALBASE}/lib
SOUND_CMAKE_ON=		-DENABLE_SOUND=1
SOUND_CMAKE_OFF=	-DENABLE_SOUND=0
LEVELDB_CMAKE_ON=	-DENABLE_LEVELDB=1
LEVELDB_CMAKE_OFF=	-DENABLE_LEVELDB=0
LEVELDB_LIB_DEPENDS=	libleveldb.so:${PORTSDIR}/databases/leveldb
SQLITE3_CMAKE_ON=	-DENABLE_SQLITE3=1
SQLITE3_CMAKE_OFF=	-DENABLE_SQLITE3=0
SQLITE3_LIB_DEPENDS=	libsqlite3.so:${PORTSDIR}/databases/sqlite3
REDIS_CMAKE_ON=		-DENABLE_REDIS=1
REDIS_CMAKE_OFF=	-DENABLE_REDIS=0
REDIS_LIB_DEPENDS=	libhiredis.so:${PORTSDIR}/databases/hiredis
LUAJIT_LIB_DEPENDS=	libluajit-5.1.so:${PORTSDIR}/lang/luajit
MINETEST_CMAKE_ON=	-DMINETEST_PROTO=1
MINETEST_CMAKE_OFF=	-DMINETEST_PROTO=0

.include <bsd.port.options.mk>

.if ${PORT_OPTIONS:MCLIENT} && ${PORT_OPTIONS:MSOUND}
USES+=		openal
LIB_DEPENDS+=	libvorbis.so:${PORTSDIR}/audio/libvorbis
.endif

.if ${PORT_OPTIONS:MSERVER}
USE_RC_SUBR=	${PORTNAME}
USERS=		minetest # reuse, as freeminer is a fork of
GROUPS=		minetest
.endif

# hacky way to not bring irrlicht and X11 depends for server only
.if ! ${PORT_OPTIONS:MCLIENT} && ${PORT_OPTIONS:MSERVER}
BUILD_DEPENDS+=	${NONEXISTENT}:${PORTSDIR}/x11-toolkits/irrlicht:patch
IRRLICHT_INCLUDE_DIR=	`${MAKE} -C ${PORTSDIR}/x11-toolkits/irrlicht -V WRKSRC`/include
CMAKE_ARGS+=	-DIRRLICHT_INCLUDE_DIR:STRING="${IRRLICHT_INCLUDE_DIR}"
EXTRA_PATCHES+=	${FILESDIR}/extra-patch-irrlichtdepend
.endif

# when 1.x.x released:
#		libjsoncpp.so:${PORTSDIR}/devel/jsoncpp
#CMAKE_ARGS+=-DENABLE_SYSTEM_JSONCPP=1 -DJSON_INCLUDE_DIR=${LOCALBASE}/include/jsoncpp

post-patch:
	@${REINPLACE_CMD} -e 's|/usr/local|${LOCALBASE}|' \
		${WRKSRC}/cmake/Modules/*.cmake
	@${REINPLACE_CMD} -e 's|-lpthread|-pthread|' \
		${WRKSRC}/src/CMakeLists.txt
.if ! ${PORT_OPTIONS:MLUAJIT}
	@${REINPLACE_CMD} -e '/find_/ s|LUA_|NO&|g' \
		${WRKSRC}/src/CMakeLists.txt
.endif

post-install:
	@${MKDIR} ${STAGEDIR}${DATADIR}/games
	@${MV} ${STAGEDIR}${PREFIX}/etc/${PORTNAME}.conf.example \
		${STAGEDIR}${PREFIX}/etc/${PORTNAME}.conf.sample

.include <bsd.port.mk>