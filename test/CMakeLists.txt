# Build program.
add_executable(strings strings.c)
add_executable(gksort gksort.c)
add_executable(fis fis.c)
add_executable(gkrw rw.c)
add_executable(gkgraph gkgraph.c)
add_executable(csrcnv csrcnv.c)
add_executable(grKx grKx.c)
add_executable(m2mnbrs m2mnbrs.c)
add_executable(cmpnbrs cmpnbrs.c)
add_executable(splatt2svd splatt2svd.c)


foreach(prog strings gksort fis gkrw gkgraph csrcnv grKx m2mnbrs cmpnbrs splatt2svd )
  target_link_libraries(${prog} GKlib)
endforeach(prog)

foreach(prog strings gksort fis gkrw gkgraph csrcnv grKx)
  target_link_libraries(${prog} GKlib)
  add_test(NAME ${prog} COMMAND ${prog})
endforeach(prog)

# Install a subset of them
install(TARGETS csrcnv RUNTIME DESTINATION bin)

# gkuniq uses asm for linux, do not build or test it for windows or mac
if (NOT WIN32 AND NOT APPLE)
  add_executable(gkuniq gkuniq.c)
  target_link_libraries(gkuniq GKlib)
  add_test(NAME gkuniq COMMAND gkuniq)
  set_tests_properties(gkuniq PROPERTIES TIMEOUT 30)
endif()