## NavViewEx is an extension of the XAML `NavigationView` control introduced in Fall Creator's Update as discussed in MSDN Magazine, December 2017

```
    public class NavViewEx : NavigationView
    {
        Frame _frame;

        public Type SettingsPageType { get; set; }
        public event EventHandler SettingsInvoked;

        public NavViewEx()
        {
            Content = _frame = new Frame();
            _frame.Navigated += Frame_Navigated;
            ItemInvoked += NavViewEx_ItemInvoked;
            SystemNavigationManager.GetForCurrentView().BackRequested += ShellPage_BackRequested;
            RegisterPropertyChangedCallback(IsPaneOpenProperty, IsPaneOpenChanged);
            Loaded += NavViewEx_Loaded;
        }

        private void NavViewEx_Loaded(object sender, RoutedEventArgs e)
        {
            if (FindStart() is NavigationViewItem i && i != null)
            {
                Navigate(i.GetValue(NavProperties.PageTypeProperty) as Type);
            }
            IsPaneOpenChanged(this, null);
            UpdateBackButton();
            UpdateHeader();
        }

        public enum HeaderBehaviors { Hide, Remove, None }

        public HeaderBehaviors HeaderBehavior { get; set; } = HeaderBehaviors.Remove;

        public void Navigate(Type type)
        {
            Navigate(_frame, type);
        }

        public new object SelectedItem
        {
            set
            {
                if (value != base.SelectedItem)
                {
                    if (value == SettingsItem)
                    {
                        if (SettingsPageType != null)
                        {
                            Navigate(SettingsPageType);
                            base.SelectedItem = value;
                            _frame.BackStack.Clear();
                        }
                        SettingsInvoked?.Invoke(this, EventArgs.Empty);
                    }
                    else if (value is NavigationViewItem i && i != null)
                    {
                        Navigate(i.GetValue(NavProperties.PageTypeProperty) as Type);
                        base.SelectedItem = value;
                        _frame.BackStack.Clear();
                    }
                }
                UpdateBackButton();
                UpdateHeader();
            }
        }

        protected virtual void Navigate(Frame frame, Type type)
        {
            frame.Navigate(type);
        }

        protected virtual void UpdateBackButton()
        {
            SystemNavigationManager.GetForCurrentView().AppViewBackButtonVisibility =
                    (_frame.CanGoBack) ? AppViewBackButtonVisibility.Visible : AppViewBackButtonVisibility.Collapsed;
        }

        private NavigationViewItem FindStart()
        {
            return MenuItems.OfType<NavigationViewItem>().SingleOrDefault(x => (bool)x.GetValue(NavProperties.IsStartPageProperty));
        }

        private void IsPaneOpenChanged(DependencyObject sender, DependencyProperty dp)
        {
            foreach (var item in MenuItems.OfType<NavigationViewItemHeader>())
            {
                switch (HeaderBehavior)
                {
                    case HeaderBehaviors.Hide:
                        item.Opacity = IsPaneOpen ? 1 : 0;
                        break;
                    case HeaderBehaviors.Remove:
                        item.Visibility = IsPaneOpen ? Visibility.Visible : Visibility.Collapsed;
                        break;
                    case HeaderBehaviors.None:
                        // empty
                        break;
                }
            }
        }

        private void NavViewEx_ItemInvoked(NavigationView sender, NavigationViewItemInvokedEventArgs args)
        {
            SelectedItem = (args.IsSettingsInvoked) ? SettingsItem : Find(args.InvokedItem.ToString());
        }

        private void Frame_Navigated(object sender, Windows.UI.Xaml.Navigation.NavigationEventArgs e)
        {
            SelectedItem = (e.SourcePageType == SettingsPageType) ? SettingsItem : Find(e.SourcePageType) ?? base.SelectedItem;
        }

        private void UpdateHeader()
        {
            if (_frame.Content is Page p && p.GetValue(NavProperties.HeaderProperty) is string s && !string.IsNullOrEmpty(s))
            {
                Header = s;
            }
        }

        private void ShellPage_BackRequested(object sender, BackRequestedEventArgs e)
        {
            if (_frame.CanGoBack)
            {
                _frame.GoBack();
            }
        }

        private NavigationViewItem Find(string content)
        {
            return MenuItems.OfType<NavigationViewItem>().SingleOrDefault(x => x.Content.Equals(content));
        }

        private NavigationViewItem Find(Type type)
        {
            return MenuItems.OfType<NavigationViewItem>().SingleOrDefault(x => type.Equals(x.GetValue(NavProperties.PageTypeProperty)));
        }
    }

    public partial class NavProperties : DependencyObject
    {
        public static Type GetPageType(NavigationViewItem obj)
            => (Type)obj.GetValue(PageTypeProperty);
        public static void SetPageType(NavigationViewItem obj, Type value)
            => obj.SetValue(PageTypeProperty, value);
        public static readonly DependencyProperty PageTypeProperty =
            DependencyProperty.RegisterAttached("PageType", typeof(Type),
                typeof(NavProperties), new PropertyMetadata(null));

        public static bool GetIsStartPage(NavigationViewItem obj)
            => (bool)obj.GetValue(IsStartPageProperty);
        public static void SetIsStartPage(NavigationViewItem obj, bool value)
            => obj.SetValue(IsStartPageProperty, value);
        public static readonly DependencyProperty IsStartPageProperty =
            DependencyProperty.RegisterAttached("IsStartPage", typeof(bool),
                typeof(NavProperties), new PropertyMetadata(false));

        public static string GetHeader(Page obj)
            => (string)obj.GetValue(HeaderProperty);
        public static void SetHeader(Page obj, string value)
            => obj.SetValue(HeaderProperty, value);
        public static readonly DependencyProperty HeaderProperty =
            DependencyProperty.RegisterAttached("Header", typeof(string),
                typeof(NavProperties), new PropertyMetadata(null));
    }
```
