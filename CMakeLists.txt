set(COMPONENT_SRCS
  driver/atomvm_dht.c
  )

set(COMPONENT_ADD_INCLUDEDIRS
  driver/include
  )

set(COMPONENT_REQUIRES driver)

register_component()
