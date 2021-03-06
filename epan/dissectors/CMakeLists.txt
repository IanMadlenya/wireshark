# CMakeLists.txt
#
# Wireshark - Network traffic analyzer
# By Gerald Combs <gerald@wireshark.org>
# Copyright 1998 Gerald Combs
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
#

ADD_CUSTOM_CMAKE_INCLUDE()

add_subdirectory(asn1 EXCLUDE_FROM_ALL)
add_subdirectory(dcerpc EXCLUDE_FROM_ALL)
add_subdirectory(pidl EXCLUDE_FROM_ALL)

include_directories(
	${CMAKE_CURRENT_BINARY_DIR}
	${CMAKE_CURRENT_SOURCE_DIR}
)

add_custom_command(
	OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/packet-ncp2222.c
	COMMAND ${PYTHON_EXECUTABLE}
		${CMAKE_SOURCE_DIR}/tools/ncp2222.py
		-o ${CMAKE_CURRENT_BINARY_DIR}/packet-ncp2222.c
	DEPENDS
		packet-ncp2222.inc
		${CMAKE_SOURCE_DIR}/tools/ncp2222.py
)

add_custom_target(x11-dissector
	COMMAND ${PERL_EXECUTABLE}
		${CMAKE_SOURCE_DIR}/tools/process-x11-fields.pl ${CMAKE_CURRENT_SOURCE_DIR}
		< ${CMAKE_CURRENT_SOURCE_DIR}/x11-fields
	COMMAND ${PERL_EXECUTABLE}
		${CMAKE_SOURCE_DIR}/tools/process-x11-xcb.pl ${CMAKE_CURRENT_SOURCE_DIR}
	DEPENDS
		x11-fields
		${CMAKE_SOURCE_DIR}/tools/process-x11-fields.pl
		${CMAKE_SOURCE_DIR}/tools/process-x11-xcb.pl
		${CMAKE_CURRENT_SOURCE_DIR}/xcbproto
		${CMAKE_CURRENT_SOURCE_DIR}/mesa
)

set_target_properties(x11-dissector PROPERTIES FOLDER "Generated Dissectors/X11")

set(CLEAN_ASN1_DISSECTOR_SRC
	packet-ansi_map.c
	packet-acp133.c
	packet-acse.c
	packet-ansi_tcap.c
	packet-atn-ulcs.c
	packet-atn-cm.c
	packet-atn-cpdlc.c
	packet-c1222.c
	packet-camel.c
	packet-cdt.c
	packet-charging_ase.c
	packet-cmip.c
	packet-cmp.c
	packet-cms.c
	packet-credssp.c
	packet-crmf.c
	packet-dap.c
	packet-disp.c
	packet-dop.c
	packet-dsp.c
	packet-ecp.c
	packet-ess.c
	packet-ftam.c
	packet-goose.c
	packet-gsm_map.c
	packet-HI2Operations.c
	packet-h225.c
	packet-h235.c
	packet-h245.c
	packet-h248.c
	packet-h282.c
	packet-h283.c
	packet-h323.c
	packet-h450.c
	packet-h450-ros.c
	packet-h460.c
	packet-h501.c
	packet-hnbap.c
	packet-idmp.c
	packet-ilp.c
	packet-inap.c
	packet-isdn-sup.c
	packet-kerberos.c
	packet-lcsap.c
	packet-ldap.c
	packet-logotypecertextn.c
	packet-lpp.c
	packet-lppa.c
	packet-lppe.c
	packet-lte-rrc.c
	packet-m2ap.c
	packet-m3ap.c
	packet-mms.c
	packet-mpeg-audio.c
	packet-mpeg-pes.c
	packet-mudurl.c
	packet-nbap.c
	packet-novell_pkis.c
	packet-ns_cert_exts.c
	packet-ocsp.c
	packet-p1.c
	packet-p22.c
	packet-p7.c
	packet-p772.c
	packet-pcap.c
	packet-pkcs1.c
	packet-pkcs12.c
	packet-pkinit.c
	packet-pkix1explicit.c
	packet-pkix1implicit.c
	packet-pkixac.c
	packet-pkixproxy.c
	packet-pkixqualified.c
	packet-pkixtsp.c
	packet-pres.c
	packet-q932.c
	packet-q932-ros.c
	packet-qsig.c
	packet-ranap.c
	packet-rnsap.c
	packet-ros.c
	packet-rrc.c
	packet-rrlp.c
	packet-rtse.c
	packet-rua.c
	packet-s1ap.c
	packet-sabp.c
	packet-sbc-ap.c
	packet-smrse.c
	packet-snmp.c
	packet-spnego.c
	packet-sv.c
	packet-t124.c
	packet-t125.c
	packet-t38.c
	packet-tcap.c
	packet-tetra.c
	packet-ulp.c
	packet-wlancertextn.c
	packet-x2ap.c
	packet-x509af.c
	packet-x509ce.c
	packet-x509if.c
	packet-x509sat.c
)

set(DIRTY_ASN1_DISSECTOR_SRC
)

set(ASN1_DISSECTOR_SRC
	${CLEAN_ASN1_DISSECTOR_SRC}
	${DIRTY_ASN1_DISSECTOR_SRC}
)

set(DIRTY_CORBA_IDL_DISSECTOR_SRC
	packet-cosnaming.c
	packet-gias.c
	packet-parlay.c
	packet-tango.c
)

set(CLEAN_CORBA_IDL_DISSECTOR_SRC
	packet-coseventcomm.c
)

set(CORBA_IDL_DISSECTOR_SRC
	${CLEAN_CORBA_IDL_DISSECTOR_SRC}
	${DIRTY_CORBA_IDL_DISSECTOR_SRC}
)

source_group(dissectors-corba FILES ${CORBA_IDL_DISSECTOR_SRC})

add_library(dissectors-corba OBJECT
	${CORBA_IDL_DISSECTOR_SRC}
)

set_target_properties(dissectors-corba PROPERTIES
	FOLDER "Libs/epan/dissectors"
	COMPILE_DEFINITIONS "WS_BUILD_DLL"
	COMPILE_OPTIONS "${WS_WARNINGS_C_FLAGS}"
)

if(WIN32 AND "${CMAKE_GENERATOR}" STREQUAL "Visual Studio 12 2013 Win64")
	# CORBA IDL dissectors are triggering a MSVC 2013 x64 /O2 bug leading to a NULL pointer dereference (see bug 12495)
	target_compile_options(dissectors-corba PRIVATE $<$<CONFIG:Release>:/Ox> $<$<CONFIG:RelWithDebInfo>:/Ox>)
endif()

set(NCP2222_DISSECTOR_SRC
	packet-ncp2222.c
)

source_group(dissectors-ncp2222 FILES ${NCP2222_DISSECTOR_SRC})

set(DISSECTOR_SRC
	file-btsnoop.c
	file-elf.c
	file-file.c
	file-gif.c
	file-jpeg.c
	file-mp4.c
	file-pcap.c
	file-pcapng.c
	file-png.c
	packet-2dparityfec.c
	packet-3com-njack.c
	packet-3com-xns.c
	packet-3g-a11.c
	packet-6lowpan.c
	packet-9p.c
	packet-a21.c
	packet-aarp.c
	packet-aastra-aasp.c
	packet-acap.c
	packet-acn.c
	packet-acr122.c
	packet-actrace.c
	packet-adb.c
	packet-adb_cs.c
	packet-adb_service.c
	packet-adwin-config.c
	packet-adwin.c
	packet-aeron.c
	packet-afp.c
	packet-afs.c
	packet-agentx.c
	packet-aim.c
	packet-ajp13.c
	packet-alcap.c
	packet-alljoyn.c
	packet-amqp.c
	packet-amr.c
	packet-ancp.c
	packet-ans.c
	packet-ansi_637.c
	packet-ansi_683.c
	packet-ansi_801.c
	packet-ansi_a.c
	packet-aodv.c
	packet-aoe.c
	packet-aol.c
	packet-ap1394.c
	packet-app-pkix-cert.c
	packet-applemidi.c
	packet-aprs.c
	packet-arcnet.c
	packet-armagetronad.c
	packet-arp.c
	packet-artnet.c
	packet-aruba-adp.c
	packet-aruba-erm.c
	packet-aruba-iap.c
	packet-aruba-papi.c
	packet-ar_drone.c
	packet-asap.c
	packet-ascend.c
	packet-asf.c
	packet-assa_r3.c
	packet-asterix.c
	packet-at.c
	packet-atalk.c
	packet-ath.c
	packet-atm.c
	packet-atmtcp.c
	packet-auto_rp.c
	packet-ax25.c
	packet-ax25-kiss.c
	packet-ax25-nol3.c
	packet-ax4000.c
	packet-ayiya.c
	packet-babel.c
	packet-bacapp.c
	packet-bacnet.c
	packet-banana.c
	packet-bat.c
	packet-batadv.c
	packet-bctp.c
	packet-beep.c
	packet-bencode.c
	packet-ber.c
	packet-bfcp.c
	packet-bfd.c
	packet-bgp.c
	packet-bitcoin.c
	packet-bittorrent.c
	packet-bjnp.c
	packet-bluecom.c
	packet-bluetooth.c
	packet-bmc.c
	packet-bmp.c
	packet-bofl.c
	packet-bootp.c
	packet-bootparams.c
	packet-bpdu.c
	packet-bpq.c
	packet-brdwlk.c
	packet-brp.c
	packet-bssap.c
	packet-bssgp.c
	packet-bt-dht.c
	packet-bt-utp.c
	packet-bt3ds.c
	packet-btamp.c
	packet-btatt.c
	packet-btbnep.c
	packet-btbredr_rf.c
	packet-btavctp.c
	packet-btavdtp.c
	packet-btavrcp.c
	packet-bthci_acl.c
	packet-bthci_cmd.c
	packet-bthci_evt.c
	packet-bthci_sco.c
	packet-bthci_vendor.c
	packet-bthcrp.c
	packet-bthfp.c
	packet-bthid.c
	packet-bthsp.c
	packet-btl2cap.c
	packet-btle.c
	packet-btle_rf.c
	packet-btmcap.c
	packet-btrfcomm.c
	packet-btsap.c
	packet-btsdp.c
	packet-btsmp.c
	packet-bvlc.c
	packet-bzr.c
	packet-c15ch.c
	packet-calcappprotocol.c
	packet-canopen.c
	packet-capwap.c
	packet-carp.c
	packet-cast.c
	packet-catapult-dct2000.c
	packet-cattp.c
	packet-cbor.c
	packet-ccsds.c
	packet-cdp.c
	packet-cell_broadcast.c
	packet-ceph.c
	packet-cfdp.c
	packet-cfm.c
	packet-cgmp.c
	packet-chargen.c
	packet-chdlc.c
	packet-cigi.c
	packet-cimd.c
	packet-cimetrics.c
	packet-cip.c
	packet-cipmotion.c
	packet-cipsafety.c
	packet-cisco-erspan.c
	packet-cisco-fp-mim.c
	packet-cisco-marker.c
	packet-cisco-metadata.c
	packet-cisco-oui.c
	packet-cisco-sm.c
	packet-cisco-ttag.c
	packet-cisco-wids.c
	packet-classicstun.c
	packet-clearcase.c
	packet-clip.c
	packet-clique-rm.c
	packet-clnp.c
	packet-cmpp.c
	packet-cnip.c
	packet-coap.c
	packet-collectd.c
	packet-componentstatus.c
	packet-cops.c
	packet-corosync-totemnet.c
	packet-corosync-totemsrp.c
	packet-cosine.c
	packet-couchbase.c
	packet-cp2179.c
	packet-cpfi.c
	packet-cpha.c
	packet-cql.c
	packet-csm-encaps.c
	packet-csn1.c
	packet-ctdb.c
	packet-cups.c
	packet-daap.c
	packet-data.c
	packet-daytime.c
	packet-db-lsp.c
	packet-dbus.c
	packet-dcc.c
	packet-dccp.c
	packet-dcerpc-bossvr.c
	packet-dcerpc-browser.c
	packet-dcerpc-cds_clerkserver.c
	packet-dcerpc-cds_solicit.c
	packet-dcerpc-conv.c
	packet-dcerpc-cprpc_server.c
	packet-dcerpc-dce122.c
	packet-dcerpc-dtsprovider.c
	packet-dcerpc-dtsstime_req.c
	packet-dcerpc-epm.c
	packet-dcerpc-fileexp.c
	packet-dcerpc-fldb.c
	packet-dcerpc-frsapi.c
	packet-dcerpc-frsrpc.c
	packet-dcerpc-ftserver.c
	packet-dcerpc-icl_rpc.c
	packet-dcerpc-krb5rpc.c
	packet-dcerpc-llb.c
	packet-dcerpc-messenger.c
	packet-dcerpc-mgmt.c
	packet-dcerpc-ndr.c
	packet-dcerpc-netlogon.c
	packet-dcerpc-pnp.c
	packet-dcerpc-rdaclif.c
	packet-dcerpc-rep_proc.c
	packet-dcerpc-roverride.c
	packet-dcerpc-rpriv.c
	packet-dcerpc-rras.c
	packet-dcerpc-rs_acct.c
	packet-dcerpc-rs_attr.c
	packet-dcerpc-rs_attr_schema.c
	packet-dcerpc-rs_bind.c
	packet-dcerpc-rs_misc.c
	packet-dcerpc-rs_pgo.c
	packet-dcerpc-rs_plcy.c
	packet-dcerpc-rs_prop_acct.c
	packet-dcerpc-rs_prop_acl.c
	packet-dcerpc-rs_prop_attr.c
	packet-dcerpc-rs_prop_pgo.c
	packet-dcerpc-rs_prop_plcy.c
	packet-dcerpc-rs_pwd_mgmt.c
	packet-dcerpc-rs_repadm.c
	packet-dcerpc-rs_replist.c
	packet-dcerpc-rs_repmgr.c
	packet-dcerpc-rs_unix.c
	packet-dcerpc-rsec_login.c
	packet-dcerpc-samr.c
	packet-dcerpc-secidmap.c
	packet-dcerpc-spoolss.c
	packet-dcerpc-svcctl.c
	packet-dcerpc-tapi.c
	packet-dcerpc-tkn4int.c
	packet-dcerpc-trksvr.c
	packet-dcerpc-ubikdisk.c
	packet-dcerpc-ubikvote.c
	packet-dcerpc-update.c
	packet-dcerpc.c
	packet-dcm.c
	packet-dcom-dispatch.c
	packet-dcom-oxid.c
	packet-dcom-remact.c
	packet-dcom-remunkn.c
	packet-dcom-sysact.c
	packet-dcom.c
	packet-dcp-etsi.c
	packet-ddtp.c
	packet-dec-bpdu.c
	packet-dec-dnart.c
	packet-dect.c
	packet-devicenet.c
	packet-dhcp-failover.c
	packet-dhcpv6.c
	packet-diameter.c
	packet-diameter_3gpp.c
	packet-diffserv-mpls-common.c
	packet-dis.c
	packet-distcc.c
	packet-dji-uav.c
	packet-dlm3.c
	packet-dlsw.c
	packet-dmp.c
	packet-dmx.c
	packet-dnp.c
	packet-dns.c
	packet-dof.c
	packet-dplay.c
	packet-dpnet.c
	packet-dpnss-link.c
	packet-dpnss.c
	packet-drda.c
	packet-dsi.c
	packet-dsr.c
	packet-dtcp-ip.c
	packet-dtls.c
	packet-dtn.c
	packet-dtp.c
	packet-dtpt.c
	packet-dua.c
	packet-dvb-ait.c
	packet-dvb-bat.c
	packet-dvb-data-mpe.c
	packet-dvb-eit.c
	packet-dvb-ipdc.c
	packet-dvb-nit.c
	packet-dvb-s2-bb.c
	packet-dvb-sdt.c
	packet-dvb-tdt.c
	packet-dvb-tot.c
	packet-dvbci.c
	packet-dvmrp.c
	packet-e100.c
	packet-e164.c
	packet-e212.c
	packet-eap.c
	packet-eapol.c
	packet-echo.c
	packet-ecmp.c
	packet-ecp-oui.c
	packet-edonkey.c
	packet-egd.c
	packet-ehdlc.c
	packet-ehs.c
	packet-eigrp.c
	packet-eiss.c
	packet-elasticsearch.c
	packet-elcom.c
	packet-elmi.c
	packet-enc.c
	packet-enip.c
	packet-enrp.c
	packet-enttec.c
	packet-epl.c
	packet-epl-profile-parser.c
	packet-epl_v1.c
	packet-epmd.c
	packet-epon.c
	packet-erf.c
	packet-erldp.c
	packet-esio.c
	packet-esis.c
	packet-etag.c
	packet-etch.c
	packet-eth.c
	packet-etherip.c
	packet-ethertype.c
	packet-etsi_card_app_toolkit.c
	packet-etv.c
	packet-evrc.c
	packet-exec.c
	packet-exported_pdu.c
	packet-extreme.c
	packet-fc00.c
	packet-fc.c
	packet-fcct.c
	packet-fcdns.c
	packet-fcels.c
	packet-fcfcs.c
	packet-fcfzs.c
	packet-fcgi.c
	packet-fcip.c
	packet-fclctl.c
	packet-fcoe.c
	packet-fcoib.c
	packet-fcp.c
	packet-fcsb3.c
	packet-fcsp.c
	packet-fcswils.c
	packet-fbzero.c
	packet-fddi.c
	packet-fefd.c
	packet-ff.c
	packet-finger.c
	packet-fip.c
	packet-fix.c
	packet-flexnet.c
	packet-flexray.c
	packet-flip.c
	packet-fmp.c
	packet-fmp_notify.c
	packet-fmtp.c
	packet-force10-oui.c
	packet-forces.c
	packet-foundry.c
	packet-fp_hint.c
	packet-fr.c
	packet-fractalgeneratorprotocol.c
	packet-frame.c
	packet-ftp.c
	packet-fw1.c
	packet-g723.c
	packet-gadu-gadu.c
	packet-gdb.c
	packet-gdsdb.c
	packet-gearman.c
	packet-ged125.c
	packet-geneve.c
	packet-gfp.c
	packet-gift.c
	packet-giop.c
	packet-git.c
	packet-glbp.c
	packet-gluster_cli.c
	packet-gluster_pmap.c
	packet-glusterd.c
	packet-glusterfs.c
	packet-glusterfs_hndsk.c
	packet-gmhdr.c
	packet-gmr1_bcch.c
	packet-gmr1_common.c
	packet-gmr1_dtap.c
	packet-gmr1_rach.c
	packet-gmr1_rr.c
	packet-gmrp.c
	packet-gnutella.c
	packet-gopher.c
	packet-gpef.c
	packet-gprs-llc.c
	packet-gprscdr.c
	packet-gre.c
	packet-gsm_a_bssmap.c
	packet-gsm_a_common.c
	packet-gsm_a_dtap.c
	packet-gsm_a_gm.c
	packet-gsm_a_rp.c
	packet-gsm_a_rr.c
	packet-gsm_abis_om2000.c
	packet-gsm_abis_oml.c
	packet-gsm_abis_tfp.c
	packet-gsm_abis_pgsl.c
	packet-gsm_bsslap.c
	packet-gsm_bssmap_le.c
	packet-gsm_cbch.c
	packet-gsm_ipa.c
	packet-gsm_osmux.c
	packet-gsm_rlcmac.c
	packet-gsm_sim.c
	packet-gsm_sms.c
	packet-gsm_sms_ud.c
	packet-gsm_um.c
	packet-gsmtap.c
	packet-gsmtap_log.c
	packet-gssapi.c
	packet-gtp.c
	packet-gtpv2.c
	packet-gvcp.c
	packet-gvrp.c
	packet-gvsp.c
	packet-h1.c
	packet-h221_nonstd.c
	packet-h223.c
	packet-h248_10.c
	packet-h248_2.c
	packet-h248_3gpp.c
	packet-h248_7.c
	packet-h248_annex_c.c
	packet-h248_annex_e.c
	packet-h248_q1950.c
	packet-h261.c
	packet-h263.c
	packet-h263p.c
	packet-h264.c
	packet-hartip.c
	packet-hazelcast.c
	packet-hci_h1.c
	packet-hci_h4.c
	packet-hci_mon.c
	packet-hci_usb.c
	packet-hclnfsd.c
	packet-hcrt.c
	packet-hdcp.c
	packet-hdcp2.c
	packet-hdfs.c
	packet-hdfsdata.c
	packet-hdmi.c
	packet-hip.c
	packet-hiqnet.c
	packet-hislip.c
	packet-hl7.c
	packet-homeplug-av.c
	packet-homeplug.c
	packet-homepna.c
	packet-hp-erm.c
	packet-hpext.c
	packet-hpfeeds.c
	packet-hpsw.c
	packet-hpteam.c
	packet-hsms.c
	packet-hsr-prp-supervision.c
	packet-hsr.c
	packet-hsrp.c
	packet-http.c
	packet-http2.c
	packet-http-urlencoded.c
	packet-hyperscsi.c
	packet-i2c.c
	packet-iana-oui.c
	packet-iapp.c
	packet-iax2.c
	packet-icap.c
	packet-icep.c
	packet-icmp.c
	packet-icmpv6.c
	packet-icp.c
	packet-icq.c
	packet-idp.c
	packet-idrp.c
	packet-iec104.c
	packet-ieee1722.c
	packet-ieee17221.c
	packet-ieee80211-netmon.c
	packet-ieee80211-prism.c
	packet-ieee80211-radio.c
	packet-ieee80211-radiotap-iter.c
	packet-ieee80211-radiotap.c
	packet-ieee80211-wlancap.c
	packet-ieee80211.c
	packet-ieee802154.c
	packet-ieee8021ah.c
	packet-ieee8023.c
	packet-ieee802a.c
	packet-ifcp.c
	packet-igap.c
	packet-igmp.c
	packet-igrp.c
	packet-imap.c
	packet-imf.c
	packet-indigocare-icall.c
	packet-indigocare-netrix.c
	packet-infiniband.c
	packet-infiniband_sdp.c
	packet-interlink.c
	packet-ip.c
	packet-ipars.c
	packet-ipdc.c
	packet-ipdr.c
	packet-iperf.c
	packet-ipfc.c
	packet-ipmi.c
	packet-ipmi-app.c
	packet-ipmi-bridge.c
	packet-ipmi-chassis.c
	packet-ipmi-picmg.c
	packet-ipmi-se.c
	packet-ipmi-session.c
	packet-ipmi-storage.c
	packet-ipmi-trace.c
	packet-ipmi-transport.c
	packet-ipmi-pps.c
	packet-ipmi-update.c
	packet-ipmi-vita.c
	packet-ipnet.c
	packet-ipoib.c
	packet-ipos.c
	packet-ipp.c
	packet-ipsec-tcp.c
	packet-ipsec-udp.c
	packet-ipsec.c
	packet-ipsi-ctl.c
	packet-ipv6.c
	packet-ipvs-syncd.c
	packet-ipx.c
	packet-ipxwan.c
	packet-irc.c
	packet-isakmp.c
	packet-iscsi.c
	packet-isdn.c
	packet-iser.c
	packet-isi.c
	packet-isis-clv.c
	packet-isis-hello.c
	packet-isis-lsp.c
	packet-isis-snp.c
	packet-isis.c
	packet-isl.c
	packet-ismacryp.c
	packet-ismp.c
	packet-isns.c
	packet-iso14443.c
	packet-iso15765.c
	packet-iso7816.c
	packet-iso8583.c
	packet-isobus.c
	packet-isobus-vt.c
	packet-isup.c
	packet-itdm.c
	packet-iua.c
	packet-iuup.c
	packet-iwarp-ddp-rdmap.c
	packet-iwarp-mpa.c
	packet-ixiatrailer.c
	packet-ixveriwave.c
	packet-j1939.c
	packet-jmirror.c
	packet-jpeg.c
	packet-json.c
	packet-juniper.c
	packet-jxta.c
	packet-k12.c
	packet-kadm5.c
	packet-kafka.c
	packet-kdp.c
	packet-kdsp.c
	packet-kerberos4.c
	packet-kingfisher.c
	packet-kink.c
	packet-kismet.c
	packet-klm.c
	packet-knet.c
	packet-knxnetip.c
	packet-kpasswd.c
	packet-kt.c
	packet-l1-events.c
	packet-l2tp.c
	packet-lacp.c
	packet-lanforge.c
	packet-lapb.c
	packet-lapbether.c
	packet-lapd.c
	packet-lapdm.c
	packet-laplink.c
	packet-lapsat.c
	packet-lat.c
	packet-lbm.c
	packet-lbmc.c
	packet-lbmpdm.c
	packet-lbmpdmtcp.c
	packet-lbmr.c
	packet-lbtrm.c
	packet-lbtru.c
	packet-lbttcp.c
	packet-ldp.c
	packet-ldss.c
	packet-lg8979.c
	packet-lge_monitor.c
	packet-link16.c
	packet-linx.c
	packet-lisp-data.c
	packet-lisp-tcp.c
	packet-lisp.c
	packet-llc.c
	packet-lldp.c
	packet-llrp.c
	packet-llt.c
	packet-lltd.c
	packet-lmi.c
	packet-lmp.c
	packet-logcat.c
	packet-logcat-text.c
	packet-lon.c
	packet-loop.c
	packet-loratap.c
	packet-lorawan.c
	packet-lpd.c
	packet-lsc.c
	packet-lsd.c
	packet-ltp.c
	packet-lwapp.c
	packet-lwm.c
	packet-lwm2mtlv.c
	packet-lwres.c
	packet-m2pa.c
	packet-m2tp.c
	packet-m2ua.c
	packet-m3ua.c
	packet-maap.c
	packet-mac-lte-framed.c
	packet-mac-lte.c
	packet-maccontrol.c
	packet-macsec.c
	packet-mactelnet.c
	packet-manolito.c
	packet-marker.c
	packet-mausb.c
	packet-mbim.c
	packet-mbtcp.c
	packet-mcpe.c
	packet-mdshdr.c
	packet-media.c
	packet-megaco.c
	packet-memcache.c
	packet-mesh.c
	packet-meta.c
	packet-metamako.c
	packet-mgcp.c
	packet-mih.c
	packet-mikey.c
	packet-mime-encap.c
	packet-mint.c
	packet-miop.c
	packet-mip.c
	packet-mip6.c
	packet-mka.c
	packet-mle.c
	packet-mmse.c
	packet-mndp.c
	packet-mojito.c
	packet-moldudp.c
	packet-moldudp64.c
	packet-mongo.c
	packet-mount.c
	packet-mp2t.c
	packet-mp4ves.c
	packet-mpeg-ca.c
	packet-mpeg-descriptor.c
	packet-mpeg-dsmcc.c
	packet-mpeg-pat.c
	packet-mpeg-pmt.c
	packet-mpeg-sect.c
	packet-mpeg1.c
	packet-mpls-echo.c
	packet-mpls-pm.c
	packet-mpls-psc.c
	packet-mplstp-oam.c
	packet-mpls-y1711.c
	packet-mpls.c
	packet-mq-base.c
	packet-mq-pcf.c
	packet-mq.c
	packet-mqtt.c
	packet-mqtt-sn.c
	packet-mrcpv2.c
	packet-mrdisc.c
	packet-mrp-mmrp.c
	packet-mrp-msrp.c
	packet-mrp-mvrp.c
	packet-ms-mms.c
	packet-msdp.c
	packet-msn-messenger.c
	packet-msnip.c
	packet-msnlb.c
	packet-msproxy.c
	packet-msrp.c
	packet-mstp.c
	packet-mswsp.c
	packet-mtp2.c
	packet-mtp3.c
	packet-mtp3mg.c
	packet-multipart.c
	packet-mux27010.c
	packet-mysql.c
	packet-nas_eps.c
	packet-nasdaq-itch.c
	packet-nasdaq-soup.c
	packet-nat-pmp.c
	packet-nb_rtpmux.c
	packet-nbd.c
	packet-nbifom.c
	packet-nbipx.c
	packet-nbt.c
	packet-ncp-nmas.c
	packet-ncp-sss.c
	packet-ncp.c
	packet-ncs.c
	packet-ndmp.c
	packet-ndp.c
	packet-ndps.c
	packet-negoex.c
	packet-netanalyzer.c
	packet-netbios.c
	packet-netdump.c
	packet-netgear-ensemble.c
	packet-netflow.c
	packet-netlink-generic.c
	packet-netlink-netfilter.c
	packet-netlink-nl80211.c
	packet-netlink-route.c
	packet-netlink-sock_diag.c
	packet-netlink.c
	packet-netperfmeter.c
	packet-netrom.c
	packet-netsync.c
	packet-nettl.c
	packet-newmail.c
	packet-nflog.c
	packet-nfs.c
	packet-nfsacl.c
	packet-nfsauth.c
	packet-nhrp.c
	packet-nisplus.c
	packet-nlm.c
	packet-nlsp.c
	packet-nntp.c
	packet-noe.c
	packet-nordic_ble.c
	packet-ns-ha.c
	packet-ns-mep.c
	packet-ns-rpc.c
	packet-nsip.c
	packet-nsh.c
	packet-nsrp.c
	packet-nstrace.c
	packet-nt-oui.c
	packet-nt-tpcp.c
	packet-ntlmssp.c
	packet-ntp.c
	packet-null.c
	packet-nvme.c
	packet-nvme-rdma.c
	packet-nwmtp.c
	packet-nwp.c
	packet-oampdu.c
	packet-obd-ii.c
	packet-obex.c
	packet-ocfs2.c
	packet-oicq.c
	packet-oipf.c
	packet-olsr.c
	packet-omapi.c
	packet-omron-fins.c
	packet-opa.c
	packet-opa-fe.c
	packet-opa-mad.c
	packet-opa-snc.c
	packet-openflow.c
	packet-openflow_v1.c
	packet-openflow_v4.c
	packet-openflow_v5.c
	packet-openflow_v6.c
	packet-opensafety.c
	packet-openthread.c
	packet-openvpn.c
	packet-openwire.c
	packet-opsi.c
	packet-optommp.c
	packet-osc.c
	packet-osi-options.c
	packet-osi.c
	packet-ositp.c
	packet-ospf.c
	packet-ossp.c
	packet-ouch.c
	packet-p_mul.c
	packet-pathport.c
	packet-packetbb.c
	packet-packetlogger.c
	packet-pagp.c
	packet-paltalk.c
	packet-pana.c
	packet-pcap_pktdata.c
	packet-pcapng_block.c
	packet-pcep.c
	packet-pcli.c
	packet-pcnfsd.c
	packet-pcp.c
	packet-pdc.c
	packet-pdcp-lte.c
	packet-peekremote.c
	packet-per.c
	packet-pflog.c
	packet-pgm.c
	packet-pgsql.c
	packet-pim.c
	packet-pingpongprotocol.c
	packet-pktap.c
	packet-pktc.c
	packet-pktgen.c
	packet-pmproxy.c
	packet-pnrp.c
	packet-pop.c
	packet-portmap.c
	packet-ppcap.c
	packet-ppi-antenna.c
	packet-ppi-geolocation-common.c
	packet-ppi-gps.c
	packet-ppi-sensor.c
	packet-ppi-vector.c
	packet-ppi.c
	packet-ppp.c
	packet-pppoe.c
	packet-pptp.c
	packet-prp.c
	packet-ptp.c
	packet-ptpip.c
	packet-pulse.c
	packet-pvfs2.c
	packet-pw-atm.c
	packet-pw-cesopsn.c
	packet-pw-common.c
	packet-pw-eth.c
	packet-pw-fr.c
	packet-pw-hdlc.c
	packet-pw-oam.c
	packet-pw-satop.c
	packet-q2931.c
	packet-q708.c
	packet-q931.c
	packet-q933.c
	packet-qllc.c
	packet-qnet6.c
	packet-quake.c
	packet-quake2.c
	packet-quake3.c
	packet-quakeworld.c
	packet-quic.c
	packet-radius.c
	packet-radius_packetcable.c
	packet-raknet.c
	packet-raw.c
	packet-rdm.c
	packet-rdp.c
	packet-rdt.c
	packet-redback.c
	packet-redbackli.c
	packet-reload-framing.c
	packet-reload.c
	packet-retix-bpdu.c
	packet-rfc2190.c
	packet-rfid-felica.c
	packet-rfid-mifare.c
	packet-rfid-pn532.c
	packet-rfid-pn532-hci.c
	packet-rftap.c
	packet-rgmp.c
	packet-riemann.c
	packet-rip.c
	packet-ripng.c
	packet-rlc-lte.c
	packet-rlc.c
	packet-rlm.c
	packet-rlogin.c
	packet-rmcp.c
	packet-rmi.c
	packet-rmp.c
	packet-rmt-alc.c
	packet-rmt-fec.c
	packet-rmt-lct.c
	packet-rmt-norm.c
	packet-rohc.c
	packet-roofnet.c
	packet-rpc.c
	packet-rpcap.c
	packet-rpcrdma.c
	packet-rpki-rtr.c
	packet-rpl.c
	packet-rquota.c
	packet-rsh.c
	packet-rsip.c
	packet-rsl.c
	packet-rstat.c
	packet-rsvd.c
	packet-rsvp.c
	packet-rsync.c
	packet-rtacser.c
	packet-rtcdc.c
	packet-rtcp.c
	packet-rtitcp.c
	packet-rtls.c
	packet-rtmpt.c
	packet-rtnet.c
	packet-rtp-events.c
	packet-rtp-midi.c
	packet-rtp.c
	packet-rtpproxy.c
	packet-rtps.c
	packet-rtsp.c
	packet-rudp.c
	packet-rwall.c
	packet-rx.c
	packet-s5066sis.c
	packet-s5066dts.c
	packet-s7comm.c
	packet-s7comm_szl_ids.c
	packet-sadmind.c
	packet-sametime.c
	packet-sap.c
	packet-sasp.c
	packet-sbus.c
	packet-sbc.c
	packet-sccp.c
	packet-sccpmg.c
	packet-scop.c
	packet-scriptingservice.c
	packet-scsi-mmc.c
	packet-scsi-osd.c
	packet-scsi-sbc.c
	packet-scsi-smc.c
	packet-scsi-ssc.c
	packet-scsi.c
	packet-scte35.c
	packet-sctp.c
	packet-sdh.c
	packet-sdlc.c
	packet-sdp.c
	packet-sebek.c
	packet-selfm.c
	packet-sercosiii.c
	packet-ses.c
	packet-sflow.c
	packet-sgsap.c
	packet-shim6.c
	packet-sigcomp.c
	packet-simple.c
	packet-simulcrypt.c
	packet-sip.c
	packet-sipfrag.c
	packet-sita.c
	packet-skinny.c
	packet-skype.c
	packet-slimp3.c
	packet-sll.c
	packet-slowprotocols.c
	packet-slsk.c
	packet-smb-browse.c
	packet-smb-common.c
	packet-smb-logon.c
	packet-smb-mailslot.c
	packet-smb-pipe.c
	packet-smb-sidsnooping.c
	packet-smb-direct.c
	packet-smb.c
	packet-smb2.c
	packet-smcr.c
	packet-sml.c
	packet-smpp.c
	packet-smtp.c
	packet-sna.c
	packet-snaeth.c
	packet-sndcp-xid.c
	packet-sndcp.c
	packet-snort.c
	packet-socketcan.c
	packet-socks.c
	packet-soupbintcp.c
	packet-spdy.c
	packet-spice.c
	packet-spp.c
	packet-spray.c
	packet-sprt.c
	packet-srp.c
	packet-srvloc.c
	packet-sscf-nni.c
	packet-sscop.c
	packet-ssh.c
	packet-ssl-utils.c
	packet-ssl.c
	packet-sstp.c
	packet-stanag4607.c
	packet-starteam.c
	packet-stat-notify.c
	packet-stat.c
	packet-stt.c
	packet-stun.c
	packet-sua.c
	packet-swipe.c
	packet-symantec.c
	packet-sync.c
	packet-synergy.c
	packet-synphasor.c
	packet-sysdig-event.c
	packet-sysex.c
	packet-syslog.c
	packet-t30.c
	packet-tacacs.c
	packet-tali.c
	packet-tapa.c
	packet-tcp.c
	packet-tcpros.c
	packet-tdmoe.c
	packet-tdmop.c
	packet-tds.c
	packet-teamspeak2.c
	packet-teimanagement.c
	packet-teklink.c
	packet-telkonet.c
	packet-telnet.c
	packet-teredo.c
	packet-text-media.c
	packet-tfp.c
	packet-tftp.c
	packet-thread.c
	packet-thrift.c
	packet-time.c
	packet-tipc.c
	packet-tivoconnect.c
	packet-tn3270.c
	packet-tn5250.c
	packet-tnef.c
	packet-tns.c
	packet-tpkt.c
	packet-tpncp.c
	packet-tr.c
	packet-trill.c
	packet-trmac.c
	packet-tsp.c
	packet-tte-pcf.c
	packet-tte.c
	packet-turbocell.c
	packet-turnchannel.c
	packet-tuxedo.c
	packet-tzsp.c
	packet-u3v.c
	packet-ua.c
	packet-ua3g.c
	packet-uasip.c
	packet-uaudp.c
	packet-ubertooth.c
	packet-ucp.c
	packet-udld.c
	packet-udp.c
	packet-udp-nm.c
	packet-uds.c
	packet-udt.c
	packet-uftp.c
	packet-uftp4.c
	packet-uhd.c
	packet-uma.c
	packet-umts_fp.c
	packet-umts_mac.c
	packet-usb-audio.c
	packet-usb-ccid.c
	packet-usb-com.c
	packet-usb-dfu.c
	packet-usb-hid.c
	packet-usb-hub.c
	packet-usb-i1d3.c
	packet-usb-masstorage.c
	packet-usb-video.c
	packet-usb.c
	packet-usbip.c
	packet-user_encap.c
	packet-userlog.c
	packet-uts.c
	packet-v120.c
	packet-v150fw.c
	packet-v52.c
	packet-v5dl.c
	packet-v5ef.c
	packet-v5ua.c
	packet-vcdu.c
	packet-vicp.c
	packet-vines.c
	packet-vlan.c
	packet-vmlab.c
	packet-vnc.c
	packet-vntag.c
	packet-vp8.c
	packet-vrrp.c
	packet-vrt.c
	packet-vsip.c
	packet-vsock.c
	packet-vssmonitoring.c
	packet-vtp.c
	packet-vuze-dht.c
	packet-vxi11.c
	packet-vxlan.c
	packet-wai.c
	packet-wap.c
	packet-wassp.c
	packet-waveagent.c
	packet-wbxml.c
	packet-wccp.c
	packet-wcp.c
	packet-websocket.c
	packet-wfleet-hdlc.c
	packet-who.c
	packet-whois.c
	packet-wifi-display.c
	packet-wifi-p2p.c
	packet-windows-common.c
	packet-winsrepl.c
	packet-wlccp.c
	packet-wol.c
	packet-wow.c
	packet-wps.c
	packet-wreth.c
	packet-wsmp.c
	packet-wsp.c
	packet-wtls.c
	packet-wtp.c
	packet-x11.c
	packet-x25.c
	packet-x29.c
	packet-xcsl.c
	packet-xdmcp.c
	packet-xip.c
	packet-xip-serval.c
	packet-xmcp.c
	packet-xml.c
	packet-xmpp-conference.c
	packet-xmpp-core.c
	packet-xmpp-gtalk.c
	packet-xmpp-jingle.c
	packet-xmpp-other.c
	packet-xmpp-utils.c
	packet-xmpp.c
	packet-xot.c
	packet-xtp.c
	packet-xyplex.c
	packet-yami.c
	packet-yhoo.c
	packet-ymsg.c
	packet-ypbind.c
	packet-yppasswd.c
	packet-ypserv.c
	packet-ypxfr.c
	packet-zbee-aps.c
	packet-zbee-nwk.c
	packet-zbee-nwk-gp.c
	packet-zbee-security.c
	packet-zbee-zcl.c
	packet-zbee-zcl-closures.c
	packet-zbee-zcl-general.c
	packet-zbee-zcl-ha.c
	packet-zbee-zcl-hvac.c
	packet-zbee-zcl-lighting.c
	packet-zbee-zcl-meas-sensing.c
	packet-zbee-zcl-misc.c
	packet-zbee-zcl-sas.c
	packet-zbee-zcl-se.c
	packet-zbee-zdp.c
	packet-zbee-zdp-binding.c
	packet-zbee-zdp-discovery.c
	packet-zbee-zdp-management.c
	packet-zebra.c
	packet-zep.c
	packet-ziop.c
	packet-zrtp.c
	packet-zvt.c
)

set(DISSECTOR_SUPPORT_SRC
	errno.c
	packet-dcerpc-nt.c
	usb.c
	snort-config.c
	register.c
)
source_group(dissector-support FILES ${DISSECTOR_SUPPORT_SRC})

set(DISSECTOR_FILES
	${DISSECTOR_SRC}
	${PIDL_DISSECTOR_SRC}
	${PIDL_DISSECTOR_AUTOGEN_SRC}
	${ASN1_DISSECTOR_SRC}
	${NCP2222_DISSECTOR_SRC}
	${CUSTOM_DISSECTOR_SRC}
	${CUSTOM_DIRTY_ASN1_DISSECTOR_SRC}
)
source_group(dissectors FILES ${DISSECTOR_FILES})

set(ALL_DISSECTOR_SRC
	${DISSECTOR_FILES}
	${CORBA_IDL_DISSECTOR_SRC}
)

register_dissector_files(register.c
	dissectors
	${ALL_DISSECTOR_SRC}
)

set(CLEAN_FILES
	${DISSECTOR_SRC}
	${PIDL_DISSECTOR_SRC}
	${CLEAN_ASN1_DISSECTOR_SRC}
	${CLEAN_CORBA_IDL_DISSECTOR_SRC}
	${NCP2222_DISSECTOR_SRC}
	${DISSECTOR_SUPPORT_SRC}
)

set_source_files_properties(
	${CLEAN_FILES}
	PROPERTIES
	COMPILE_FLAGS "${WERROR_COMMON_FLAGS}"
)

add_library(dissectors OBJECT
	${DISSECTOR_FILES}
	${DISSECTOR_SUPPORT_SRC}
)

set_target_properties(dissectors PROPERTIES
	FOLDER "Libs/epan/dissectors"
	COMPILE_DEFINITIONS "WS_BUILD_DLL"
	COMPILE_OPTIONS "${WS_WARNINGS_C_FLAGS}"
)

set(CHECKAPI_FILE_LIST "${CMAKE_CURRENT_BINARY_DIR}/files.txt")
file(GLOB DISSECTOR_HEADERS RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" "*.h")
file(WRITE "${CHECKAPI_FILE_LIST}" "${DISSECTOR_FILES}" ";" "${CORBA_IDL_DISSECTOR_SRC}")
CHECKAPI(
	NAME
	  dissectors
	SWITCHES
	  -g abort -g termoutput -build --file ${CHECKAPI_FILE_LIST}
	SOURCES
	  ${DISSECTOR_SUPPORT_SRC}
	  ${DISSECTOR_HEADERS}
)

#
# Editor modelines  -  http://www.wireshark.org/tools/modelines.html
#
# Local variables:
# c-basic-offset: 8
# tab-width: 8
# indent-tabs-mode: t
# End:
#
# vi: set shiftwidth=8 tabstop=8 noexpandtab:
# :indentSize=8:tabSize=8:noTabs=false:
#
