if get_option('examples')
  examples = [
    'list_ports',
    'port_info',
    'port_config',
    'await_events',
    'handle_errors',
    'send_receive'
  ]

  foreach example : examples
    executable(example,
      example + '.c',
      link_with: libserialport,
      include_directories: inc,
      install: false
    )
  endforeach
endif
